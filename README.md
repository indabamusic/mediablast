# mediablast
Generic processing server built on [node-plan](https://github.com/superjoe30/node-plan).

## Quick Start

See a [working example of a mediablast server](https://gist.github.com/4312843)

### Server Code

```js
var http = require('http');
var mediablast = require('mediablast');

var app = mediablast({
  settingsFile: 'settings.json'
});

app.registerTask('s3.store', require('node-plan-s3-upload'));
app.registerTask('s3.retrieve', require('node-plan-s3-download'));
app.registerTask('audio.waveform', require('node-plan-waveform'));
app.registerTask('audio.transcode', require('node-plan-transcode'));
app.registerTask('meta.callback', require('node-plan-callback'));

var server = http.createServer(app);
server.listen(function() {
  console.log("mediablast server online");
});
```

### HTTP Client Usage

#### Admin

Currently mediablast only supports one user. By default this user name is `admin`
and the password is `3pTkHwHV`. You should change this before you deploy.

For better user account management, see [#1](https://github.com/superjoe30/mediablast/issues/1).

#### Managing Processing Templates

##### Creating

A template is a specification of what will be done with the user input.

You can create a template by making POST request to `/admin/templates/create`
with HTTP basic auth and a JSON payload:

```json
{
  "options": {
    "task-1": {
      // task-1 options that apply to every instance of task-1 below.
      // this is a convenience that can be overridden by the task
      // instance options.
      "a": 1,
      "b": 2
    }
  },
  "tasks": {
    "example-1": {
      "task": "task-1",
      "options": {
        // these options override the global options above
        "a": 3
      }
    },
    "example-2": {
      "task": "task-2",
      "dependencies": [ "example-1" ]
    }
  }
}
```

A successful response looks like:

```json
{
  "success": true,
  "template": {
    "id": "6373af69-367e-48f4-8734-f30cee4f7541",
    "options": {
      "task-1": {
        "a": 1,
        "b": 2
      }
    },
    "tasks": {
      "example-1": {
        "task": "task-1",
        "options": {
          "a": 3
        }
      },
      "example-2": {
        "task": "task-2",
        "dependencies": [ "example-1" ]
      }
    }
  }
}
```

An error response looks like:

```json
{
  "success": false,
  "error": "InvalidInput"
}
```

##### Deleting

To delete a template, make a POST request to `/admin/templates/delete` with
HTTP basic auth.

```
POST /admin/templates/delete
Content-Type: application/json

{
  "templateId": "6373af69-367e-48f4-8734-f30cee4f7541"
}
```

#### Submitting a Job

To submit a job, make a `POST` request to `/`. The tasks you have registered
dictate what parameters to send with this request. Usually you will want
this request to be a multipart file upload.

The request will look something like:

```json
{
  "templateId": "6373af69-367e-48f4-8734-f30cee4f7541"
}
```

The response will look like:

```json
{
  "id": "20fff6bc-fa45-44c2-a92b-7df597fdb8b1",
  // range: [0, 1]
  "progress": 1,
  // one of ['processing', 'complete']
  "state": "complete",
  // error identifier. If this is truthy, the job completed with an error.
  "error": null,
  "startDate": "2012-08-27T13:41:17.253Z",
  "endDate": "2012-08-27T13:41:22.006Z",
  "exports": {
    "example-1": {
      // Each task has its own documentation which describes how its `exports`
      // object is constructed.
    },
    "example-2": {
      ...
    },
  }
}
```


### Admin Interface

Hit `/admin` with your browser to monitor all jobs in the system.
