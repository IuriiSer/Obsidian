[Source](https://alvinlal.netlify.app/blog/single-thread-vs-child-process-vs-worker-threads-vs-cluster-in-nodejs)

## child processes

The `child_process` module provides the ability to spawn new processes which has their own memory. The communication between these processes is established through IPC (inter-process communication) provided by the operating system.

There are mainly three methods inside this module that we care about.

-   `child_process.spawn()`
-   `child_process.fork()`
-   `child_process.exec()`

### child_process.spawn()

This method is used to spawn a child process asynchronously. This child process can be any command that can be run from a terminal.

spawn takes the following syntax:- spawn("comand to run","array of arguments",optionsObject)

The optionsObject have a variety of settings which can be found in official nodejs [documentation](https://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options).

The code below spawns an ls (list directory) process with arguments `-lash` and the directory name from query strings and sends its output back.

**childspawnServer.js**
```javascript
const express = require("express")
const app = express()
const { spawn } = require("child_process") //equal to const spawn = require('child_process').spawn

app.get("/ls", (req, res) => {
  const ls = spawn("ls", ["-lash", req.query.directory])
  ls.stdout.on("data", data => {
    //Pipe (connection) between stdin,stdout,stderr are established between the parent
    //node.js process and spawned subprocess and we can listen the data event on the stdout

    res.write(data.toString()) //date would be coming as streams(chunks of data)
    // since res is a writable stream,we are writing to it
  })
  ls.on("close", code => {
    console.log(`child process exited with code ${code}`)
    res.end() //finally all the written streams are send back when the subprocess exit
  })
})

app.listen(7000, () => console.log("listening on port 7000"))
```
_[see code on github](https://github.com/alvinlal/blogRepos/blob/master/01-single_thread-vs-child_process-vs-worker_threads-vs-cluster-in-nodejs/childspawnServer.js)_

[![ls-output](https://alvinlal.netlify.app/static/d58d5b5b605da07190ec96b175d2e5e1/5ece7/lsoutput.png "ls-output")](https://alvinlal.netlify.app/static/d58d5b5b605da07190ec96b175d2e5e1/a2c9b/lsoutput.png)

Nothing is stopping us from spawning a nodejs process and doing another task there, but `fork()` is a better way to do so.

### child_process.fork()

`child_process.fork()` is specifically used to spawn new nodejs processes. Like spawn, the returned childProcess object will have built-in IPC communication channel that allows messages to be passed back and forth between the parent and child.

fork takes the following syntax:- fork("path to module","array of arguments","optionsObject")

The optionsObject have a variety of settings which can be found in official nodejs [documentation](https://nodejs.org/api/child_process.html#child_process_child_process_fork_modulepath_args_options).

Using fork(), we can solve the problem discussed [above](https://alvinlal.netlify.app/blog/single-thread-vs-child-process-vs-worker-threads-vs-cluster-in-nodejs#theproblem) by forking a separate nodejs process and executing the function in that process and return the answer to the parent process whenever it is done. In that way, the parent process won't be blocked and can continue responding to requests.

**childforkServer.js**

```javascript
const express = require("express")
const app = express()
const { fork } = require("child_process")

app.get("/isprime", (req, res) => {
  const childProcess = fork("./forkedchild.js") //the first argument to fork() is the name of the js file to be run by the child process
  childProcess.send({ number: parseInt(req.query.number) }) //send method is used to send message to child process through IPC
  const startTime = new Date()
  childProcess.on("message", message => {
    //on("message") method is used to listen for messages send by the child process
    const endTime = new Date()
    res.json({
      ...message,
      time: endTime.getTime() - startTime.getTime() + "ms",
    })
  })
})

app.get("/testrequest", (req, res) => {
  res.send("I am unblocked now")
})

app.listen(3636, () => console.log("listening on port 3636"))
```

_[see code on github](https://https//github.com/alvinlal/blogRepos/blob/master/01-single_thread-vs-child_process-vs-worker_threads-vs-cluster-in-nodejs/childforkServer.js)_

***forkedchild.js**
```javascript
process.on("message", message => {
  //child process is listening for messages by the parent process
  const result = isPrime(message.number)
  process.send(result)
  process.exit() // make sure to use exit() to prevent orphaned processes
})

function isPrime(number) {
  let isPrime = true

  for (let i = 3; i < number; i++) {
    if (number % i === 0) {
      isPrime = false
      break
    }
  }

  return {
    number: number,
    isPrime: isPrime,
  }
}
```

_[see code on github](https://github.com/alvinlal/blogRepos/blob/master/01-single_thread-vs-child_process-vs-worker_threads-vs-cluster-in-nodejs/forkedchild.js)_

![app not hanging when large number is given](https://alvinlal.netlify.app/ba40c98cb3b342478ecb39fdf6764115/nonblockingprime.gif)

Hurray! our app is no longer blocking

## Caveats

1.  Separate memory is allocated for each child process which means that there is a time and resource overhead.