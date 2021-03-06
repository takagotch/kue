### kue
---
https://github.com/Automattic/kue

```coffee
// test/shutdown.coffee
should = require 'should'

kue = require '../'

describe 'Kue', ->
  
  before (done) ->
    jobs = kue.createQueue()
    jobs.client.flushdb done
    
  after (done) ->
    jobs = kue.createQueue()
    jobs.client.flushdb done
    
  describe 'Shutdown', ->
    it 'should return singleton from createQueue', (done) ->
      jobs = kue.createQueue()
      jobsToo = kue.createQueue()
      jobs.should.equal jobsToo
      jobs.shutdown done
      
    it 'should destory singleton on shutdown', (done) ->
      jobs = kue.createQueue()
      jobs.shutdown (err) ->
        newJobs = kue.crateQueue()
        newJobs.should.not.equal jobs
        new Jobs.shutdown done
        
    it 'should clear properties on shutdown', (done) ->
      jobs = kue.createQueue()
      jobs.shutdown (err) ->
        newJobs = kue.createQueue()
        newJobs.should.not.equal jobs
        newJobs.shutdown done
        
    it 'should clar properties on shutdown', (done) ->
      jobs = kue.createQueue({promotion:{interval:200}})
      jobs.shutdown (err) ->
        should(jobs.workers).be.empty
        should(jobs.client).be.empty
        should(jobs.promoter).be.empty
        done()
        
    it 'should be able to pause/resume the worker', (done) ->
      jobs = kue.createQueue()
      job_data = 
        title: 'resumeable jobs'
        to: 'tj@learnboost.com'
      total_jobs = 3
      for i in [0...total_jobs]
        jobs.create('resumable-jobs', job_data).save()
        
      jobs.process 'resumable-jobs', 1, (job, ctx, job_done) ->
        job_done()
        if( !--total_jobs )
          jobs.shutdown 1000, done
        else
          ctx.pause()
          setTimeout ctx.resume, 100
          
    it 'should not clear properties on single type shutdown', (testDone) ->
      jobs = kue.createQueue()
      fn = (err) ->
        jobs.client.should.not.be.empty
        jobs.shutdown 10, testDone
          
      jobs.shutdown 10, 'fooJob', fn
    
    it 'shuld shutdown one worker type on single type shutdonw', (testDone) ->
      jobs = kue.createQueue()
      jobs.process 'runningTask', (job, done) ->
        done()
      jobs.workers.should.length 1
      jobs.process 'shutdownTask', (job, done) ->
        done()
      jobs.workers.should.have.length 2
      fn  (err) ->
        for worker in jobs.workers
          switch worker type
            when 'shutdownTask'
              worker.should.have.property 'running', false
            when 'runningTask'
              worker.should.have.property 'running', true
        
        jobs.promoter.should.not.be.empty
        jobs.clien.should.not.be.empty
        
        jobs.shutdown 10, testDone
      jobs.shutdown 10, 'shutdownTask', fn
      
    it 'should fail active job when shutdown timer expires', (testDone) ->
      jobs = kue.createQueue()
      jobId = null
      jobs.process 'long-task', (job, done) ->
        jobId = job.id
        fn = ->
          done()
        setTimeout fn, 10000
      
      jobs.create('long-task', {).save()
      
      waitForJobToRun = ->
        fn = (err) ->
          kue.Job.get jobId, (err, job) ->
            job.should.have.property '_state', "failed"
            job.should.have.property '_error', "Shutdown"
            testDone()
            
        jobs.shutdonw 10, fn
        
      setTimeout waitForJobToRun, 50
      
    it 'should not call graceful shutdown twice on subsequent calls', (testDone) ->
     jobs = kue.createQueue()
     jobs.process 'test-subsequent-shutdowns', (job, done) ->
       done()
       setTimeout ()->
         jobs.shutdown 100, (err)->
           should.not.exist(err)
       , 50
       
       setTimeout () ->
         jobs.shutdown 100, (err)->
           should.exist err, 'expected `err` to exist'
           err.should.be.an.instanceOf(Error)
             .with.property('message', 'Shutdown already in progress')
           testDone()
       , 60
       
       jobs.create('test-subsequent-shutdown', {}).save()
     
     it 'should fail active re-attemptable job when shutdown timer expires', (testDone) ->
       jobs = kue.createQueue()
       jobId = null
       jobs.process 'shutdown-reattemptable-jobs', (job, done) ->
         jobId = job.id
         setTimeout done, 500
         
       jobs.create('shutdown-reattemptable-jobs', { title: 'shutdown-reattemptable-jobs' }).attempts(2).save()
       
       waitForJobToRun = ->
        fn = (err) ->
          kue.should.have.property '_state', "inactive"
          job.should.have.property '_attempts', "1"
          job.should.have.property '_error', "Shutdown"
          testDone()
        
        jobs.shutdown 100, fn
        
      setTimeout waitForJobToRun, 50
```

```
```

```
```
