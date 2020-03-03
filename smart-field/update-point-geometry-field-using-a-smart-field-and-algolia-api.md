---
description: >-
  Create smart field to edit point field using the address widget and algolia
  API
---

# Update point geometry field using a smart field and algolia api

Here I need to fill in 2 fields in my db for a location: address 1 \(a string\) and location \(a postresql geography point\). Although a widget with autocomplete exists to fill an address string in the UI \([https://docs.forestadmin.com/documentation/reference-guide/fields/customize-your-fields/edit-widgets\#address](https://docs.forestadmin.com/documentation/reference-guide/fields/customize-your-fields/edit-widgets#address)\), the location coordinates can only be obtained manually by looking up the address in our search engine which is not optimal.

**Objective: be able to update both fields in one input using if possible the address autocomplete.**

Approach chosen: create a smart field \(virtual field that does not exist in DB but is displayed and editable in Forest\) in your Forest Admin backend app that will serve as the input field. See how to set a smart field [here](https://docs.forestadmin.com/documentation/reference-guide/fields/create-and-manage-smart-fields). _**NB: the code above would be located at forest/events.js in my lumber app.**_  
The smart field, that I named “location setter”, is a string and returns the value of the address field for display purposes. I added the address edit widget to this field to ensure that upon edit it gave me the algolia autocomplete feature.  
Upon edit of the field, the input provided will be used to fetch the address coordinates through an API \(I used algolia places as our widget relies on their API - so I know for sure the address string passed will return the correct result from the API\) and update both the field address and the field location with the relevant values.  
Therefore as shown in the video example you can put the address and location fields in read only and edit solely the “location setter” field to set them both every time [https://recordit.co/2Tj4TDtgeo](https://recordit.co/2Tj4TDtgeo).

{% code title="forest/events.js" %}
```javascript
const Liana = require('forest-express-sequelize');
const algoliasearch = require('algoliasearch');
const places = algoliasearch.initPlaces(process.env.PLACES_APP_ID, process.env.PLACES_API_KEY);

Liana.collection('events', {
  fields: [{
    field: 'Location setter',
    type: 'String',
    get: (event) => {
      return event.address
    },
    set: (event, query) => {
      async function getLocationCoordinates(query){
          try {
            const location = await places.search({query: query, type:'address'});
            console.log('search location coordinates result', location.hits[0]._geoloc);
            return location.hits[0]._geoloc
          } catch (err) {
            console.log(err);
            console.log(err.debugData);
          }
      }

      async function setEvent(event, query) {
        const coordinates = await getLocationCoordinates(query)
        event.address = query
        console.log('new address', event.address)
        event.locationGeo = `{"type": "Point", "coordinates": [${coordinates.lat}, ${coordinates.lng}]}`
        console.log('new location', event.locationGeo)
        return event
      }

      return setEvent(event, query)
    }
  }],
});
```
{% endcode %}

