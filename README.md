# Postman-Polling Logic

This is a very simple Postman .json collection export that holds sample wait/polling logic I wrote that allows you to wait for certain conditions before advancing on in the runner flow. Note, this will only work when executing tests via the Runner, or through Newman.

Simply import this collection into your Postman instance to update and run.

This collection holds 2 requests, where the second request will only run once the first request has achieved the correct value. We can see the structure of the collection is:

![image](https://github.com/oisin-mcl/Postman-PollingLogic/assets/56615317/a49389cd-0781-495b-a6a9-35c8d7564da6)

The fist request in the collection is named "Wait Until Repsonse Returns", and uses a sample Postman GET API. As such, there is no Body needed, and no Pre-request either. The wait logic will run after the request is returned, so it is held in the Test tab.

The first thing we do is just good Postman practices - test the response code, and store the response:

![image](https://github.com/oisin-mcl/Postman-PollingLogic/assets/56615317/5bebcd70-92e9-4d28-b8dd-f239570cc679)

Next comes the beginning of the wait logic. First we set the variables for the number of loops we want, and the time between the loops:

![image](https://github.com/oisin-mcl/Postman-PollingLogic/assets/56615317/8df115a5-4efc-435d-afec-76ab97a0a02f)

Here, we set the number of potential loops to be 60, with a millisecond wait time of 1000. This results in a possible 1 minute total wait time. You can adjust these to your needs, but if CI/CD integration is your goal, add a wait of 1+ seconds to avoid unwanted memory usage. Too little of a wait can also result in values caching between responses so it may take longer overall if you avoid waits. 

After setting these, we then do a check:

![image](https://github.com/oisin-mcl/Postman-PollingLogic/assets/56615317/ce9315b8-a396-425e-a494-5b7e9d69af66)

Similar to 'for loops', we will use a counter. In this case, we will be using a collection variable as our loop counter, but environment or global variables work too. The above check will ensure that if the "loop" variable doesnt exist, meaning it is a first loop iteration, then it will create it and set it to 1. If the request is already looping, then this check is skipped.

Next, we define the wait logic. In this case, we use 3 statements in our 'If' statement, but you can add more else statements if needed.

![image](https://github.com/oisin-mcl/Postman-PollingLogic/assets/56615317/42879ad6-7fd6-4874-87de-474321d40bdc)

This is the opening statement of our If block, and is our "success" check. It will check that the "args" object is returned in the response with some parameters. If this is the case, it will set the next request to be "Next Request",, resetting the loop to 1, and outputting a console message, stopping the execution of our logic and moving on to the next request. This means the values we want are returned, and we can then move on in our collection.

However, if the "args" object is not returned with parameters, it will move on to this statement:
![image](https://github.com/oisin-mcl/Postman-PollingLogic/assets/56615317/94c014e4-67ba-4c9c-9f39-f06ef148fa6d)

Here, it checks that if the "args" object is still blank, and our counter is not above the number of loops we defined at the beginning. If this is the case, it increments the loop counter by 1, wait for the timeout we set initially, and then setting the next request to be the current request, essentially causing a loop.

The final statement in our If will only call if the request times out:
![image](https://github.com/oisin-mcl/Postman-PollingLogic/assets/56615317/6a3714ff-c4f0-4358-b165-880d82748f68)

And this will just end the flow.

This simple code block will allow us to define what conditions we wish to wait for, then loop the current request as many times as we like, as frequently as we like, until the desired output is achieved. This is something that becomes essential when testing APIs that involve other processese or async methods. With this sample code, I have been using very basic checks, but this can get as complex as you like, and involve as many extra checks as you need. 
