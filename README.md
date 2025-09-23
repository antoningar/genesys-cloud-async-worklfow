# genesys-cloud-async-worklfow
A Genesys Async Workflow is a way of doing async tasks to let customers continue their journeys while tasks are executed. This is a 100% native Genesys Cloud method.

# How it works
An initial flow will launch another process which will run independently (a workflow here, to stay in a native Genesys Cloud environment).  
Participant datas of the conversation will be the support of a bi-directionnal communication for both flows.  
The conversation id is send by the initial flow in the workflow request body.  
Then both flows are able to communicate by reading and editing participant datas of the current conversation.  

# Sample - AI intention recongition
By getting the STT from the caller in Genesys, you want your own IA to do the intention recognition.  
However with a long sentences, a recognition can take more thab 5 secondes.  
Instead of ask te caller to wait all this time, with an asynchronous workflow you use this seconds to interact with the caller.  

## Timeline - AI intention recongition
### 1. Catch caller intention
Intention is catch as text fron a Genesys Cloud inbound call flow.  
### 2. Launch recognition workflow
The initial flow execute the dedicated workflow thanks to a REST call POST /api/v2/flow/execution/{workflowId}, with the conversationId and the caller intention in the body.
### 3. Recognition workflow talk to AI
The workflow execute an REST call to the target AI asking for an intention recognition of based on caller's input.  
It updates his status to ONGOING in conversation participant datas. 
### 4. Customer journey continue
Inbound call flow inform the caller an event is organize by the company next saturday for a charity.  
Then it check in the participant datas if intention is found, but status is still ONGOING.  
An other message with the more details about the event (hours, activities...).
### 5. Workflow get AI response
AI return claler intentions.  
Workflow add the result in conversation's participant datas and set is status to SUCCESS.
### 6. Customer journey can go throught the queue
Inbound call flow see workflow status in SUCCESS.  
Intention is catch, and the call can go to the corresponding queue.

# Sample - Multiple external calls
The Alpha company customer service's IVR  need to identify callers thanks to their calling number.  
Alpha company just bought two other concurrent, Beta and Gamma. The three customer databases merge is planned for next year.  
To be able to identify the caller, now, I need to do three HTTP rest call to three different REST webservices.  

## Timeline - Multiple external calls
### 1. Inbound flow Launch the workflow
The first step of the Genesys inbound flow is a REST call POST /api/v2/flow/execution/{workflowId}, with the conversationId and the caller number in the body.    
This REST call  is a Genesys Cloud internal call, it will be extremly fast.  
Then the caller continue his journey throught the call flow by asking the caller the reason of his call.  
### 2. Workflow start it execution
Workflow is starting his execution.  
Setting his status to ONGOING by updating a conversation caller participant data.
The first REST calls done with caller number is not finding any result.
### 3. Inbound flow need the caller identity
To get the caller identity, the flow is looking for the workflow status throught his own participant datas.  
Status is ONGOING.  
The caller continue his journey trhought the call flow by asking if this is the first time the caller issue occur.
### 4. Workflow found caller's identity
Second api calls failed again, but the third hit a result.  
Workflow add caller's identity into participant datas.  
Finaly the workflow is setting his status to SUCCESS by updating the same participant data than earlier.
### 5. Inbound flow get caller's identity
To get the caller identity, the flow is looking for the workflow status throught his own participant datas.  
Status is SUCCESS.  
Flow get caller identity thanks to the others participant datas.  
Custom user experience can begin.

# Synchronous process
![](docs/synchronous.png)

# Aynchronous process
![](docs/asynchronous.png)