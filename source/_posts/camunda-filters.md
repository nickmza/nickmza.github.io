---
title: Custom filters in Camunda Tasklist
tags:
  - workflow
  - camunda
category:
  - Software Engineering
excerpt: How to add custom filters to the Camunda Tasklist.
date: 2024-02-21 00:00:00
---


Camunda Tasklist allows you to create filters allowing you to group tasks based on specific criteria. For example you could create a filter for 'Expense Approvals that are older than 5 days' or 'Any task that is assigned to me'. Whilst there are numerous options when creating filters via the UI it has one specific gap: creating filters based on Process Variables. Here is how you can accomplish this via code. 

{% codeblock lang:java %}

@Component
public class FilterBuilder {

    @Autowired
    TaskService taskService;

    @Autowired
    FilterService filterService;

    @Autowired
    AuthorizationService authorizationService;

    @EventListener(ApplicationReadyEvent.class)
    public void createCustomFilter() {

        TaskQuery myTasksQuery = taskService.createTaskQuery()
                .processVariableValueEquals("MyCustomKey","Custom");

        Filter myTasksFilter = createFilter("My Custom Tasks", myTasksQuery);

    }

    private Filter createFilter(String filterName, TaskQuery myTasksQuery) {
        Filter myTasksFilter = filterService.newTaskFilter(filterName);
        myTasksFilter.setOwner("mcbcam");
        myTasksFilter.setQuery(myTasksQuery);
        filterService.saveFilter(myTasksFilter);
        return myTasksFilter;
    }
}

{% endcodeblock %}

In your Camunda project create a new Component. Add a method to create your custom filter and decorate it with the 'ApplicationReadyEvent' Event Listener. This will trigger your code once the SpringBoot application has fully started and all the dependencies are available for injection.

Start by describing the filter's criteria. Here I am selecting for processes where the 'MyCustomKey' Process Variable has the value of 'Custom'. 

> For a full list of options to create a filter see the [TaskQuery](https://docs.camunda.org/javadoc/camunda-bpm-platform/7.3/org/camunda/bpm/engine/task/TaskQuery.html) documentation.

Then use the resulting TaskQuery object to create a new Task Filter.

At this point you will have a new filter displayed in your Tasklist.

What if you want to do this for everyone though? This can be done by creating a new Authorization for the filer that gives everyone Read permissions. Here's the code...

{% codeblock lang:java %}

    private void configureSecurity(Filter filter) {
        var authorization = authorizationService.createNewAuthorization(Authorization.AUTH_TYPE_GLOBAL);

        authorization.setUserId(Authorization.ANY);
        authorization.setResource(Resources.FILTER);
        authorization.addPermission(Permissions.READ);
        authorization.setResourceId(filter.getId());

        authorizationService.saveAuthorization(authorization);
    }

{% endcodeblock %}

There you have it. Custom filters based on Process Variables.