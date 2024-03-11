---
title: Improving Camunda UI with Form.io
tags:
  - workflow
  - camunda
category:
  - Software Engineering
excerpt: >-
  I wanted to see if I could leverage the Form.io Form Renderer from within a
  Camunda Embedded Form. Turns out you can...
date: 2024-02-21 00:00:00
ogimage: cf4.png
---


In the Camunda space there are several options for adding a user interface to your workflow. On the one extreme is the Camunda Tasklist with its Forms Modeller or Embedded Forms. On the other is a bespoke User Interface created using the technology of your choice. There are benefits and challenges with both. Whilst Tasklist allows for rapid prototyping and validation of your workflow it comes at the expense of User Experience. The Forms Modeller is not sufficient for anything but the simplest of forms. Embedded forms allow for more flexibility but creating complex forms can be time consuming. A custom application gives you full control over the user experience - but at the cost of more development effort and the added complexity of building a truely data-driven application. This last part is crucial to get right otherwise when your workflow changes your application's UI will need to be changed. 

My preference is that we always start with the simplest UI in Tasklist and rapidly iterate so that we can get the workflow and integrations in place. If at that time the UI is adequate we can move on. If not we can then try to address any UI concerns via Embedded Forms. If this fails though we are looking at having to create a fully-custom UI. The challenge we face is that the effort to create a better experience with Embedded Forms is often significant. This leads to pressure to jump straight to the fully-custom option. I wanted to see if there was a way to get a better experience out of Tasklist by addressing some of the issues with Embedded Forms.

After some searching online I came across [Form.io](https://form.io/) - a javascript-based forms engine.

# Form.io

Form.io is designed to render forms from a JSON definition and submit them to the Form.io backend for processing. However it also allows you to simply capture the data and then implement the backend yourself. In this case I wanted to use Form.io to capture my data and then pass this to Camunda. 

I started by designing a simple form using the Form.io Form Builder:
<img src="cf1.png"/>

This results in the following JSON:

{% codeblock lang:json %}
{
    "display": "form",
    "components": [
    {
        "label": "First Name",
        "applyMaskOn": "change",
        "tableView": true,
        "validate": {
        "required": true,
        "maxLength": 15
        },
        "key": "firstName",
        "type": "textfield",
        "input": true
    },
    {
        "label": "Last Name",
        "applyMaskOn": "change",
        "tableView": true,
        "validate": {
        "required": true,
        "maxWords": 20
        },
        "key": "lastName",
        "type": "textfield",
        "input": true
    }
    ]
}
{% endcodeblock %}

# Adding Form.io to Camunda Tasklist
The first thing you need to do is to add the Form.io scripts to the Tasklist. You can do this by adding the reference as a < script /> tag in the Embedded Form but this will cause issues as the script will be loaded asynchronously potentially causing a race condition. Also adding the script in this manner means that Camunda will reload the script every time the form us accessed. A better way is to provide the script to all forms. To do this create a new directory at '<Camunda>/app/tasklist/scripts/formio' and add the formio.full.min.js script. Then edit the config.js in the scripts directory as follows:

{% codeblock lang:json %}

export default {
    ...
   customScripts: [
      'scripts/formio/formio.full.min.js'
   ],
   ...
}

{% endcodeblock %}

# Creating the Embedded Form

Start by adding the Form.io stylesheets and creating a div to render the form into. Next create a function to return your JSON. You can load this from your Deployment or code it in your form. Lastly use createForm method of the Formio object to render the form.

Here is the mark=up:

{% codeblock lang:html %}
<form role="form" name="form">
  
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.0/dist/css/bootstrap.min.css" crossorigin="anonymous">
  <link rel="stylesheet" href="https://cdn.form.io/formiojs/formio.form.min.css" crossorigin="anonymous">
  <link rel="stylesheet" href="https://cdn.form.io/formiojs/formio.full.min.css" charset="utf-8">

  <div id="formio"></div>

  <script>

    function getFormData(){
        //return your Form.io JSON
    }

    Formio.createForm(document.getElementById('formio'), getFormData());

  </script>


</form>
{% endcodeblock %} 

Here is the form in Tasklist:
<img src="cf2.png"/>


# Submitting the form
To submit the form we need to get the values from the Form.io form and save them as Camunda variables. I've implemented this code in the CamScript submit function as follows:

{% codeblock lang:javascript %}
camForm.on('submit', function(evt) {

    const form = Object.values(Formio.forms)[0];

    var firstName = form.getComponent('firstName').getValue();
    var lastName = form.getComponent('lastName').getValue();
  
    camForm.variableManager.createVariable({
        name: 'firstName',
        type: 'String',
        value: firstName
    });
  
    camForm.variableManager.createVariable({
        name: 'lastName',
        type: 'String',
        value: lastName
    });
    
  });

{% endcodeblock %}

# Triggering validation 

As you can see in the screenshot above both fields are required. Form.io will validate the fields if you edit their contents but we need a way to trigger the form validation if the user clicks the 'Complete' button directly. This turned out to be a lot trickier than I expected...

To trigger Form.io validation you can simply call the 'submit' method of the 'form' object. This will throw an error detailing any validation errors and trigger rendering of those errors on the form. For example:

<img src="cf3.png"/>

The first challenge here is that 'submit' returns a promise, so we need to make so changes to accommodate this.

{% codeblock lang:javascript %}
camForm.on('submit', async function(evt) {

    const form = Object.values(Formio.forms)[0];

    try{
        await form.submit();

        var firstName = form.getComponent('firstName').getValue();
        var lastName = form.getComponent('lastName').getValue();
  
        camForm.variableManager.createVariable({
            name: 'firstName',
            type: 'String',
            value: firstName
        });
  
        camForm.variableManager.createVariable({
            name: 'lastName',
            type: 'String',
            value: lastName
        });
    }
    catch(e){
        evt.submitPrevented = true;
    }
    
  });

{% endcodeblock %}

I've added the async keyword to the event handler declaration and called 'submit' using await. If 'submit' throws an error we cancel the submit by setting 'submitPrevented' to true.

Only one problem... this does not work. Regardless of how or where I set 'submitPrevented' Camunda would not cancel the submit and move the workflow to the next activity.

The culprit is the introduction of the promise-based 'submit' method. It seems that regardless of what is inside the camForm event it is processed synchronously. I found a support article which references this behaviour and suggested a possible solution - call the event recursively. This works,  I am really not happy with it, but until I find a better solution here it is:

{% codeblock lang:javascript %}

< script cam-script type="text/form-script">

  var blockSubmit = true;

  camForm.on('submit', async function(evt) {

    evt.submitPrevented = blockSubmit;

    const form = Object.values(Formio.forms)[0];

    try{
      await form.submit();

      blockSubmit = false;

      var firstName = form.getComponent('firstName').getValue();
      var lastName = form.getComponent('lastName').getValue();

      createVariable('firstName', firstName);
      createVariable('lastName', lastName);

      $scope.complete();

    }
    catch(e){
      console.log(e);
    }
    
  });

{% endcodeblock %}

By default 'blockSubmit' is false which means that Camunda will not process the submit. When we run the 'form.submit()' if there are no issues we set 'blockSubmit' to true and then force another submit via '$scope.complete()'.

> Open to suggestions on how to make this less cringe-inducing. Let me know in the comments.

# Next Steps
This took a bit of reading and tinkering to get working but the result is a powerful forms engine running within Embedded Forms for a handful of lines of code. This should provide us more headroom in our Workflow projects between the default Tasklist experiences and needing a full-blown custom application. I'm going to try implement one of our more complex forms using Form.io and see how it goes. Will save that for another post...
