# Display a JSON

{% code title="template.hbs" %}
```javascript
{{#each-in currentRecord.forest-Request.forest-skills as |category categorySkills|}}
	{{category}}: 
	{{#each-in categorySkills as |index skill|}}
		{{if index ' - '}}{{skill.value}}
	{{/each-in}}
{{/each-in}}
```
{% endcode %}

