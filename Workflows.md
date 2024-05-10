# Launching IdentityIQ Workflows Programatically: A Comprehensive Guide

# Introduction

In the realm of IdentityIQ, the orchestration of workflows stands as a cornerstone for effective identity governance. Yet, mastering the art of workflow automation entails more than just leveraging built-in functionalities. It necessitates the integration of custom logic through rules or plugins, empowering developers to tailor IdentityIQ to their organization's unique needs, leveraging unit tests, async workflow launching at runtime, scheduling in custom tasks, etc.

# Understanding IdentityIQ Workflows Launching

IdentityIQ workflows serve as the backbone of identity governance, facilitating the automation of key processes such as access provisioning, certification, and lifecycle management. Workflows, behind the scenes, are launched in IdentityIQ functionalities recieving a Map of arguments that should be consumed by the workflow. It's important to remember that if you have any Identity based trigger requirements, like: Manager Transfer, Identity creation, Native Change, or any other any specified [here](https://documentation.sailpoint.com/identityiq/help/lcm/how_to_create_lifecycle_.html "Lifecycle Events") you should be using [Lifecycle Events](https://documentation.sailpoint.com/identityiq/help/lcm/how_to_create_lifecycle_.html "Lifecycle Events") instead.

But, as we know that OOTB functionalities don't always fit perfectly in complex IAM environments, let's talk about how to do this in a more flexible way.

# Integrating Custom Logic with IdentityIQ Workflows

Before getting deep on methods for Launching Workflows programatically, it's important considering using IIQ Console [Workflows commands](https://documentation.sailpoint.com/identityiq/help/iiqconsole/84consolecommands.html#Workflow:~:text=ShowGroup-,Workflow%20Commands,-Workflow "Workflows commands") if your requirements are: Workflow Launch with Shell or Batch Scripts, Manual Tests or other CLI integrations. With that being said, let's dive into it.

At the heart of workflow customization lies the ability to launch workflows via BeanShell Rules or Java Plugins. By embedding custom code within IdentityIQ, developers can extend its functionality and address specific business requirements with precision and efficiency.

## Step-by-Step Guide to Launching Workflows via Rules

Now that you know if launching Workflows via Java code is the best option to you. Let's understand how to do it in practice.

The Workflow Sample that we're going to use is a simple one. It's only function is to recieve an argument called "name" and log it in the screen.

Sample Workflow
```xml
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE Workflow PUBLIC "sailpoint.dtd" "sailpoint.dtd">
<Workflow explicitTransitions="true" name="Workflow Launch Test" type="LCMProvisioning">
  <Variable input="true" name="name"/>
  <Step icon="Start" name="Start" posX="20" posY="20">
    <Transition to="Generic Step"/>
  </Step>
  <Step icon="Stop" name="Stop" posX="185" posY="20"/>
  <Step icon="Default" name="Generic Step" posX="102" posY="17">
    <Arg name="name" value="ref:name"/>
    <Script>
      <Source>log.warn("Executing Workflow");

log.warn("Name provided: " + name);</Source>
    </Script>
    <Transition to="Stop"/>
  </Step>
</Workflow>
```
Sample Workflow in the Business Processes Tab
**IMAGE


### Synchronous Launch

If you want to launch synchronously a workflow and get the result at runtime, in the most native possible way, the following code is probably the right option to you. Here we are leveraging the **sailpoint.service.WorkflowService** a native IdentityIQ class not shown in the IIQ JavaDoc. This class can be instantiated recieving **sailpoint.api.SailPointContext** as an argument (line 30), and passing a prepared Map containing a single entry, with the key "workflowArgs" and the value of a **sailpoint.object.Attributes<String, Object>** (line 37).

BeanShell Rule Example
```java
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE Rule PUBLIC "sailpoint.dtd" "sailpoint.dtd">
<Rule language="beanshell" name="Launch Workflow" type="Workflow">
  <Source>
import java.util.HashMap;
import java.util.Map;
import java.util.List;

import sailpoint.object.Attributes;
import sailpoint.object.WorkflowLaunch;
import sailpoint.object.TaskResult;

import sailpoint.api.SailPointContext;
import sailpoint.tools.Message;
import sailpoint.service.WorkflowService;
import org.apache.log4j.Logger;

  // Instantiating Logger
  Logger log = Logger.getLogger("sailpoint.custom.Rule.LaunchWorkflow");
  
  
  // Method containing all Custom Logic to be passed
  public void launchWorkflow(String workflowName, Attributes attributes, SailPointContext context) throws Exception {

    // Prepare arguments to be provided
    Map preparedArgsMap = new HashMap();
    preparedArgsMap.put("workflowArgs", attributes);

    // Calls SailPoint native WorkflowService
    WorkflowService wfService = new WorkflowService(context);

    WorkflowLaunch launch = null;

    try {

      // Launch the workflow passing the arguments
      launch = wfService.launch(workflowName, preparedArgsMap);

    } catch (Exception e) {

      // Do your custom error treatment
      // ...

    }

    
    // Get information about the execution
    log.warn(launch.getStatus());
    log.warn(launch.getWorkflowName());
    log.warn(launch.getMessages());
    log.warn(launch.getVariables());
    log.warn(launch.getTaskResult());
	
  }
  
  
 // Calling the method
 Map arguments = new HashMap();
 arguments.put("name", "Moises Estevao");
 
  
 Attributes attributes = new Attributes();
 attributes.setMap(arguments);
  
 launchWorkflow("Workflow Launch Test", attributes , context);
 
  </Source>
</Rule>

```
Or if you want to do it in a Java Class instead:

Java Class Example

```java
package sailpoint.your.pack.service;

import java.util.HashMap;
import java.util.Map;
import java.util.List;

import sailpoint.object.Attributes;
import sailpoint.object.WorkflowLaunch;
import sailpoint.object.TaskResult;

import sailpoint.api.SailPointContext;

import sailpoint.tools.Message;

import sailpoint.service.WorkflowService;

public class CustomWorkflowService {
	
	// Method containing all Custom Logic to be passed
    public void launchWorkflow(Attributes<String, Object> attributes, String workflowName, SailPointContext context) throws Exception {

        // Prepare arguments to be provided
        Map<String, Object> preparedArgsMap = new HashMap<>();
        preparedArgsMap.put("workflowArgs", attributes);

        // Calls SailPoint native WorkflowService
        WorkflowService wfService = new WorkflowService(context);

        WorkflowLaunch launch = null;

        try {

            // Launch the workflow passing the arguments
            launch = wfService.launch(workflowName, args);

        } catch (Exception e) {

            // Do your custom error treatment
            // ...

        }


        // Get information about the execution
        String launchStatus = launch.getStatus();
        String launchWorkflowName = launch.getWorkflowName();
        List<Message> messages = launch.getMessages();
        Map<String, Object> variables = launch.getVariables();
        TaskResult taskResult = launch.getTaskResult();

    }
}
``` 
As Java and BeanShell are pretty similar and Java can leverage pretty much any resource presented in Rules, from now on in this document we are only going to show BeanShell Rules. If you wanna know more about Java Plugins, check out [IdentityIQ Plugins](https://community.sailpoint.com/t5/IdentityIQ-Plugins/ct-p/Plugins "IdentityIQ Plugins")

When launching the example Rule through Debug Page, we can see that the result was the expected, and a TaskResult was created in the IIQ database. 
** IMAGE

Some good things to be aware of are that the method WorkflowService.launch often does not execute the Workflow, as it is expected to be validated before being called. Remember, this is not an SDK, we are leveraging a native method, so there's a need for a previous validation verifying if the arguments are either null or void, and if the Workflow with the specified name exists.

### Another Synchronous Launch (Easiest)

In this scenario, we are leveraging **sailpoint.api.Workflower.launch(WorkflowLaunch)** to launch a Workflow. This is the easiest way to execute a workflow programatically, in a fashion "SDK way of life". Also, in this scenario we can set some other configurations in WorkflowLaunch object, such as WorkflowCase name (line 21).


```java
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE Rule PUBLIC "sailpoint.dtd" "sailpoint.dtd">
<Rule language="beanshell" name="Launch Workflow 2" type="Workflow">
  <Source>
import java.util.HashMap;

import sailpoint.api.Workflower;
import sailpoint.object.WorkflowLaunch;

// Workflow name
String wf = "Workflow Launch Test";

// Arguments section
HashMap launchArgsMap = new HashMap();
launchArgsMap.put("name", "Moises Estevao");

// Create WorkflowLaunch and set values
WorkflowLaunch wflaunch = new WorkflowLaunch();
wflaunch.setWorkflowName(wf);
wflaunch.setWorkflowRef(wf);
wflaunch.setCaseName("Random Case Name");
wflaunch.setVariables(launchArgsMap);

// Create Workflower and launch workflow from WorkflowLaunch
Workflower workflower = new Workflower(context);
WorkflowLaunch launch = workflower.launch(wflaunch);

  
  </Source>
</Rule>

```
The result of this scenario is pretty much the same from the other one, as shown in the image below:
*** IMAGE

###  Asynchronous Launch

Maybe you have an asynchronous requirement, and you don't want to use Tasks at all. What you need is a solution that triggers a Workflow in a particular time, and want to do this programatically. Then the following might help you.

This code block is using a different approach. The Rule uses **import sailpoint.api.RequestManager** to leverage a kind of queue, where a  **RequestDefinition** is sent with a time in millis to be executed and other standard configurations.


```java 
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE Rule PUBLIC "sailpoint.dtd" "sailpoint.dtd">
<Rule language="beanshell" name="Launch Workflow Asynchronously" type="Workflow">
  <Source>
import java.util.HashMap;
import java.util.Map;
import java.util.List;

import sailpoint.object.Attributes;
import sailpoint.object.WorkflowLaunch;
import sailpoint.object.TaskResult;
import sailpoint.object.Request;
import sailpoint.object.RequestDefinition;
import sailpoint.api.RequestManager;
 
import sailpoint.api.SailPointContext;
import sailpoint.tools.Message;
import sailpoint.service.WorkflowService;
import org.apache.log4j.Logger;

  
//Instantiating Logger
Logger log = Logger.getLogger("sailpoint.custom.Rule.LaunchWorkflow");
  
String workflowName = "Workflow Launch Test";
String caseName     = "Run '" + workflowName;
  
// Define the delay (in milliseconds) for starting the workflow
long delayInMillis = 5000;
long launchTime = System.currentTimeMillis() + delayInMillis;

// Append the timestamp to the case name to ensure uniqueness
caseName += "(" + launchTime + ")";

// Build the map of arguments for the Request Scheduler
Attributes reqArgs = new Attributes();

reqArgs.put(sailpoint.workflow.StandardWorkflowHandler.ARG_REQUEST_DEFINITION, sailpoint.request.WorkflowRequestExecutor.DEFINITION_NAME);
reqArgs.put(sailpoint.workflow.StandardWorkflowHandler.ARG_WORKFLOW, workflowName);
reqArgs.put(sailpoint.workflow.StandardWorkflowHandler.ARG_REQUEST_NAME, caseName);
reqArgs.put("requestName", caseName);

// Build the map of arguments for the Workflow case when it launches
Attributes wfArgs = new Attributes();
wfArgs.put("name", "Moises Estevao");
reqArgs.putAll(wfArgs);

// Use the Request Launcher to schedule the workflow request
Request req = new Request();
RequestDefinition reqdef = context.getObject(RequestDefinition.class, "Workflow Request");
req.setDefinition(reqdef);
req.setEventDate(new Date(launchTime));
req.setName(caseName);
req.setAttributes(reqdef, reqArgs);

// Schedule the workflow via the request manager
RequestManager.addRequest(context, req);
  
  	
  
  </Source>
</Rule>
```
After executing this Rule, Workflows are put in a RequestManager queue (line 57), where they'll be waiting to be processed. After 5 seconds, the result was pretty much the same from the others, as shown in the following image:
** IMAGE


# Conclusion

In the ever-evolving landscape of identity governance, the ability to automate workflows stands as a flexible asset for requirements of organizations. By harnessing the power of custom rules and plugins, developers can unlock new realms of possibility within IdentityIQ, tailored to meet the unique demands of their environment. Most IdentityIQ installations only require Workflow launching after some project maturity, this is because this feature is most commonly used to reuse custom features that have been developed over time.
