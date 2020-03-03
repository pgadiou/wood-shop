# Belongsto clickable

{% code title="template.hbs" %}
```javascript
{{#each records as |record|}}
  <BetaLinkTo
    @text={{record.forest-Request.forest-title}}
    @type='primary'
    @size='normal'
    @underline={{false}}
    @routeName='rendering.data.collection.list.viewEdit.details'
    @routeParameters={{array this.requestCollectionId record.forest-Request.id}}
    @class='my-class'
  />
{{/each}}

import Component from '@ember/component';
import SmartViewMixin from 'client/mixins/smart-view-mixin';
import { computed } from '@ember/object';
export default Component.extend(SmartViewMixin, {
  requestCollectionId: computed(function () {
    return this.getCollectionId('Request');
  }),
});
```
{% endcode %}

