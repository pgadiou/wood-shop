# Add algolia API

`npm install --save algoliasearch`

Define the following variables obtained from your Algolia account in your .env file

```text
PLACES_APP_ID = xxx
PLACES_API_KEY = xxxx
```

Add the following to be able to call the functions of the Algolia API

```text
const algoliasearch = require('algoliasearch');
const places = algoliasearch.initPlaces(process.env.PLACES_APP_ID, process.env.PLACES_API_KEY);
```

