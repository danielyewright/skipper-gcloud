[![npm version](https://img.shields.io/npm/v/skipper-gcloud.svg)](https://www.npmjs.org/package/skipper-gcloud)


skipper-gcloud
===========

A skipper adapter to allow uploading files to Google Cloud Storage


## Usage

```js
req.file('file').upload({
  adapter: require('skipper-gcloud'),
  projectId: 'project-id',
  keyFilename: '/path/to/keyfile.json',
  email: 'YOUR_GCS_EMAIL',
  scopes: ['YOUR_SCOPES'],
  bucket: 'YOUR_GCS_BUCKET',
  public: true,
}, ...);
```

## License
[MIT](./LICENSE)
