# Background Jobs Explained.
###### By Aziz Tinwala
---
Background jobs are considered as an vital concept if we want to build a large and scalable application because they take Time/CPU consuming task to a background process and hence speeding up the request/response lifecycle of a web request.

In this article, we will learn when to and how to implement background jobs in your application.

> NOTE: We will be using [node.js](https://nodejs.org/en/) for code examples given below.  Although, Background job is not limited to any one programming language.

Consider an example where we to want to upload a file to Amazon’s S3, in this case we are not sure of how much time it will take to upload the file. We cannot keep the browser waiting for response for too long as it can result in a `GATEWAY_TIMEOUT` error, what we can do is take the file upload job and add it to a background process, now our file is getting uploaded in background and we can now send an appropriate response to the browser and end the web request. This was one of the many cases where background jobs can come in handy. 

Also, there is a good practice which is to avoid web request that runs longer than 500ms. If your application has a request which probably run longer than one or two seconds, then you should consider using a background job instead.

---
Now, to implement background jobs in node.js application effectively, I have curated a list of top 3 Job Managers/Schedulers which are as follows:
* [Bull](https://github.com/OptimalBits/bull)
* [Agenda](https://github.com/rschmukler/agenda)
* [Kue](https://github.com/Automattic/kue)

This are packages easily available to download using npm. Although, `Bull` and `Kue` requires [`redis`](http://redis.io/download) and `Agenda` requires [`mongoDB`](https://www.mongodb.com/download-center) as a prerequisite. So you will need to install given prerequisites depending on the package you are using.

---
As `Bull` being my personal favourite, I will show small code snippet to setup `bull` in your node.js application.

we will need `redis` version higher or equal than 2.8.11 up and running.

### Quick Start:

`npm install bull`

```javascript
var Queue = require('bull');

var uploadQueue = Queue('file uploading', 6379, '127.0.0.1');
var emailQueue = Queue('send email', 6379, '127.0.0.1');
 
uploadQueue.process(function(job, done){
 
  // job.data contains the custom data passed when the job was created 
  // job.jobId contains id of this job. 
 
  // upload file asynchronously and report progress 
  job.progress(42);
 
  // call done when finished 
  done();
 
  // or give a error if error 
  done(Error('error uploading'));
 
  // or pass it a result 
  done();
 
  // If the job throws an unhandled exception it is also handled correctly 
  throw (Error('some unexpected error'));
});
 
emailQueue.process(function(job){
  // Processors can also return promises instead of using the done callback 
  return sendEmailProcessor();
}
 
uploadQueue.add({file: 'http://example.com/video1.mov'});
emailQueue.add({/* email object... */});
```

Almost all job managers/schedulers creates a queue of jobs to process. In above case, We are first initializing two queues uploadQueue and emailQueue. And than we add our job into the queue.

> NOTE: Node.js also provide a way to implement background jobs using [child process](https://nodejs.org/api/child_process.html)

I hope, you understood the concept and importance of Background Jobs.
