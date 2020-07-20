[![npm version](https://img.shields.io/npm/v/npm.svg)](https://www.npmjs.org/package/skipper-gcloud)

# skipper-gcloud

> Not to be confused with [skipper-gclouds](https://www.npmjs.com/package/skipper-gclouds) psst! notice the "`s`"

A skipper adapter to allow uploading files to Google Cloud Storage

## Creating a service account

You could create a new service account with the required roles for your needs and generate a key file for that service account to use in your application. You can do this by following these steps.

1. Navigate to 'APIs & Services' > 'Credentials'

2. Click on 'Create credentials' > 'Service account key'

3. You have the option of selecting a service account that already exists, or creating a new service account (for the purpose of these steps I'll select 'create new service account')

4. Select a role that allows the access to the storage you require, for example 'Storage Admin' which allows read and write operations to the bucket(s).

5. Choose either JSON or P12 (JSON is recommended) and then hit 'Create'. The key file for the service account will be downloaded to your local machine.

The key file generated in the above steps can be used to allow application to assume the identity of the service account, and therefore it's authorisation/authentication privileges.

## Usage

```js
req.file('file').upload({
  adapter: require('skipper-gcloud'),
  ...sails.config.uploads.gcloud
}, ...);
```

## Using sails-hook-uploads

```js
module.exports = {
  friendlyName: "Upload thing",

  description:
    "Upload info about an item that will be visible for friends to borrow.",

  files: ["photo"],

  inputs: {
    photo: {
      description: "Upstream for an incoming file upload.",
      type: "ref"
    }
  },

  exits: {
    success: {
      outputDescription: "The newly created `Thing`.",
      outputExample: {}
    },

    noFileAttached: {
      description: "No file was attached.",
      responseType: "badRequest"
    },

    tooBig: {
      description: "The file is too big.",
      responseType: "badRequest"
    }
  },

  fn: async function({ photo }) {
    var url = require("url");
    var util = require("util");

    // Upload the image.
    var info = await sails
      .uploadOne(photo, {
        maxBytes: 3000000,
        ...sails.config.uploads.gcloud
      })
      // Note: E_EXCEEDS_UPLOAD_LIMIT is the error code for exceeding
      // `maxBytes` for both skipper-disk and skipper-s3.
      .intercept("E_EXCEEDS_UPLOAD_LIMIT", "tooBig")
      .intercept(
        err => new Error("The photo upload failed: " + util.inspect(err))
      );

    if (!info) {
      throw "noFileAttached";
    }

    // Create a new "thing" record.
    var newThing = await Thing.create({
      imageUploadFd: info.fd,
      imageUploadMime: info.type,
      label,
      owner: this.req.me.id
    }).fetch();

    var imageSrc = url.resolve(
      sails.config.custom.baseUrl,
      "/api/v1/things/" + newThing.id + "/photo"
    );

    // Return the newly-created thing, with its `imageSrc`
    return {
      id: newThing.id,
      imageSrc
    };
  }
};
```

If using you're [sails-hook-uploads](https://www.npmjs.com/package/sails-hook-uploads), configure `config.uploads.adapter` to reference this

```js
module.exports.uploads = {
  adapter: require("skipper-gcloud")
  // dirpath: '.tmp/uploads',
  gcloud: {
    adapter: require("skipper-gcloud") // adapter tied for gcloud only
    projectId: "project-id",
    keyFilename: "key-file.json", // We assume that `key-file.json` is at the root of your sails folder
     email: "YOUR_GCS_EMAIL",
    scopes: ["YOUR_SCOPES"],
    bucket: "sails-bucket",
    public: true
  }
};
```

## License

[MIT](./LICENSE)
