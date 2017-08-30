# citizeng
A turnkey username-password framework to create an systematically-authenticated HTTP-server. 
## Purpose
This project provides a parent HTTP-server class users can derive from to build authenticated Web services. It is supposed to be used on a **small** and **local network**. A particular attention is paid to ensure it can run easily on small devices (more particularly on a [Raspberry pi](https://www.raspberrypi.org/) system).
## Features
* Easy to integrate : just concentrate on your server design!
* Use [Passport](http://passportjs.org/) as an authentication method (*'passport-local'* strategy).
* Support real-time HTML rendering thanks to [socket.io](https://socket.io/) framework.
* Simple and lightweight users management using a [sqlite3](https://www.npmjs.com/package/sqlite3) database.
* Offer basic users management: list, add, remove and password change.
## Overview
### What does the framework provide?
The framework provides only tree static routes: `/login`, `/admin` and `/404`.

* The `/login` route presents a basic username/password login interface. This is where unhauthenticated users will be redirected to. Here the user could connect to the server and/or change its password on the fly.

* The `/admin` route presents a elementary users administration interface. This is where authorized users will be able to interact with the users database.

* The `/404` route provides a fordidden access page when the user try to access an non-authorized service (because it does not exist or because the user does not hav the right to access it).

> Note : Theses routes are said to be **reserved**.

The framework provides a way to dynamically render services too thanks to the `socket.io` framework. The following routes are used for internal use:

* The `connect`()/`disconnect` routes are use to respectively open/close the `socket.io` connection.

* The `userpresent`, `userlist`, `useradd`, `userdelete` and `userchangepassword` are used to manage users database.

> Note : Theses routes are said to be **reserved**, too.

### Users type
This framework considers tree types of users, depending on their given access right :
1. `basic` user : it is the default user. It will access default services. It may not be able to reach user administration service (`/admin` route).
2. `super` user : it is an administrator user. It can reach `/admin` route.
3. `master` user : even if is the top-level-ever user type, it remains anecdotic because it is simply a `super` user (declared when inializing the server) that cannot be removed during the session.

> Note: A `basic` user can be removed from the users database by any `super` user. The `master` user cannot be removed from the users database.
## Installation
### Default installation
```Shell
npm install citizeng
```
### Installation for development
```Shell
# Clone the repository.
git clone https://github.com/LordHadder/citizeng.git mydir

# Go the cloned repository.
cd mydir

# Install required modules.
npm install express passport passport-local body-parser express-session sqlite3 socket.io --save

# Run the demo.
node citizeng-demo
```
## How to use
First, you need to load `citizeng` module to retrieve an HTTP-server object:
```JavaScript
var citizeng = require('citizeng');
```
The server object must be initialized first by providing a `port number`, a master `user name` and and the master user `password`.

In the following sample, the server will be accessible from the URL **http://localhost:3030** by the `master` user **Groot** with password **root**:
```JavaScript
citizeng.init(3030, "Groot", "root");
```
To access services, just make express-like GET and POST requests. Ths sample render a specific HTML file depending on if the user is a `basic` or a `super` user:
```JavaScript
citizeng.get("/", function (req, res) {
  // Super user's page.
  if (req.user.super) {
    res.sendFile((__dirname + "/indexsuper.html"));
  }
  // Default user's page.
  else {
    res.sendFile((__dirname + "/index.html"));
  }
});
```
Declaring a **reserved** route may have no effect.
```JavaScript
citizeng.get("/admin", function(req, res) {
  res.send("This may have not effect!");
});
```
The following example shows how a to use the `socket.io` layer to render dynamically a service.
```JavaScript
/* On client side, Emit a simple message. */
var socket = io.connect();
socket.emit("consolemessage", "Bonjour !!");

/* On server side, receive the message and just print. */
citizeng.ioset("consolemessage", function(data, ackfunc) {
  // Log.
  console.log(data);
});
```
Declaring a **reserved** socket.io route may have no effect.
```JavaScript
citizeng.ioset("useradd", function(data, ackfunc) {
  // Log.
  console.log("This may have not effect!");
});
```
Now start the server:
```JavaScript
citizeng.run();
```
## Next things to do
- [ ] To make unit test ([mocha](https://mochajs.org/)?)
- [ ] To support secured protocol (HTTPS)
- [ ] To offer a way to stylize `/login`, `/admin` and `/404` HTML rendering (generic CSS)
## License
[MIT](https://github.com/socketio/socket.io/blob/master/LICENSE)
