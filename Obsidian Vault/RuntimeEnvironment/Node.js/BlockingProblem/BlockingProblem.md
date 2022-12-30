[Source](https://alvinlal.netlify.app/blog/single-thread-vs-child-process-vs-worker-threads-vs-cluster-in-nodejs)

# The problem

Doing Input-Output bound operations such as responding to an Http request, talking to a database, talking to other servers are the areas where a Nodejs application shines. This is because of its single-threaded nature which makes it possible to handle many requests quickly with low system resource consumption. But Doing CPU bound operations like calculating the Fibonacci of a number or checking if a number is prime or not or heavy machine learning stuff is gonna make the application struggle because node only uses a single core of your CPU no matter how many cores you have.

If we are running this heavy CPU bound operation in the context of a web application,the single thread of node will be blocked and hence the webserver won't be able to respond to any request because it is busy calculating our big Fibonacci or something.

## Wait, Cant Promises solve this problem?

This may be a stupid doubt but For a brief moment, while I was researching for this article, I thought "Isnt promises supposed to solve this problem?, Isnt promises supposed to unblock stuff by doing things asynchronously?". Well yes, but no.

[![yesbutnomeme](https://alvinlal.netlify.app/static/35acef21802767a5dc2c74354cd44b3b/1bed9/yesbutnomeme.png "yesbutnomeme")](https://alvinlal.netlify.app/static/35acef21802767a5dc2c74354cd44b3b/1bed9/yesbutnomeme.png)

I found a [question](https://stackoverflow.com/questions/53876344/correct-way-to-write-a-non-blocking-function-in-node-js) on stackoverflow with the same doubt.

The promises won't help 'cause even though promises are being run asynchronously the promise executor function(our prime or not function) is called synchronously and will block our app. The reason why promises are glorified in the javascript community as a way to do "asynchronous non-blocking operations" is because they are good at doing jobs that take more time, but not more CPU power. By "doing jobs that take more time" I meant jobs like talking to a database, talking to another server, etc which is 99% of what web servers do. These jobs are not immediate and will take relatively more time. Javascript promises accomplish this by pushing the job to a special queue and listening for an event (like a database has returned with data) to happen and do a function (often referred to as a "callback function") when that event has happened. but hey, won't another thread will be required to listen for that event? , Yes it does. How node manages this queues,events and listening threads internally can be a separate article of itself .

# The solution

Node js provides three solutions for solving this problem

-  [[child_processes]]
-  cluster
-  [[worker_threads]]

## What solution is better

**Cluster:** 

- Built on top of Child_process, but with TCP distributed between clusters.
- Best for distributing/balancing incoming http requests, while bad for cpu intensive tasks.
- Works by taking advantage of available cores in cpu, by cloning nodeJS webserver instances on other cores.

**Child_process:**

- Make use also of different cores available, but its bad since it costs huge amount of resources to fork a child process since it creates virtual memory.
- Forked processes could communicate with the master thread through events and vice versa, but there is no communication between forked processes.

**Worker threads:**

- Threads use the shared memory space.
- Same as child process, but forked processes can communicate with each other using `bufferArray`