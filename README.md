# App Dev - Developing a Backend Service: Node.js
We will
- Create a Cloud Pub/Sub topic.
- Publish messages to the topic.
- Subscribe to the topic to receive messages in a separate worker application.
- Use the Cloud Natural Language Machine Learning API.
- Create a Cloud Spanner database instance.
- Develop a Cloud Spanner database schema.
- Insert data into a Cloud Spanner database.

Below is an example of pub/sub design application
![1](https://user-images.githubusercontent.com/73010204/142826868-d8f37caf-4ec5-48a6-969b-2517d5cc66de.png)

## Activate Google Cloud Shell
Google Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Google Cloud Shell provides command-line access to your GCP resources.</br>
In GCP console, on the top right toolbar, click the Open Cloud Shell button.</br>
![a](https://user-images.githubusercontent.com/73010204/142588938-f700d742-d45d-46a8-a0e8-d5a89a8d6c01.png)</br>
It takes a few moments to provision and connect to the environment. When you are connected, you are already authenticated, and the project is set to your PROJECT_ID. </br>
**gcloud** is the command-line tool for Google Cloud Platform. It comes pre-installed on Cloud Shell and supports tab-completion.</br>
You can list the active account name with this command:
```sh
gcloud auth list
```
Output:</br>
![b](https://user-images.githubusercontent.com/73010204/142589185-f2aef8e8-0f3f-4bce-bfdd-ca66c2049f1b.PNG)</br>
You can list the project ID with this command:
```sh
gcloud config list project
```
Output:</br>
![c](https://user-images.githubusercontent.com/73010204/142589426-9649e06a-e57f-47ed-9d8a-de2610534f19.PNG)</br>
## Clone source code in Cloud Shell
In Cloud Shell, clone the repository for the class.
```sh
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
```
Create a soft link as a shortcut to the working directory.
```sh
ln -s ~/training-data-analyst/courses/developingapps/v1.2/nodejs/pubsub-languageapi-spanner ~/pubsub-languageapi-spanner
```
## Configure and run the case study application
Navigate to the directory that contains the smaple files for this lab:
```sh
cd ~/pubsub-languageapi-spanner/start
```
To configure the application, execute the following command:
```sh
. prepare_web_environment.sh
```
This script file
- Creates an App Engine application.
- Exports environment variables: GCLOUD_PROJECT and GCLOUD_BUCKET.
- Runs npm install.
- Creates entities in Cloud Datastore.
- Prints out the GCP Project ID.

Here it is
```sh
echo "Creating App Engine app"
gcloud app create --region "us-central"

echo "Making bucket: gs://$DEVSHELL_PROJECT_ID-media"
gsutil mb gs://$DEVSHELL_PROJECT_ID-media

echo "Exporting GCLOUD_PROJECT and GCLOUD_BUCKET"
export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media

echo "Installing dependencies"
npm install -g npm@6.11.3
npm update

echo "Creating Datastore entities"
node setup/add_entities.js

echo "Project ID: $DEVSHELL_PROJECT_ID"
```
To run the application, execute the following command:
```sh
npm start
```
To add a new Cloud Shell session, click **Open a new tab (+)** on the right of the Cloud Shell tab.</br>
To change the working directory, enter the following command:
```sh
cd ~/pubsub-languageapi-spanner/start
```
To prepare the environment in the second Cloud Shell window, enter the following command:
```sh
. prepare_worker_environment.sh
```
This script file:
- Exports environment variables GCLOUD_PROJECT and GCLOUD_BUCKET.
- Creates and configures a GCP Service Account.
- Prints out the GCP Project ID.

Here it is
```sh
echo "Exporting GCLOUD_PROJECT and GCLOUD_BUCKET"
export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media

echo "Creating quiz-account Service Account"
gcloud iam service-accounts create quiz-account --display-name "Quiz Account"
gcloud iam service-accounts keys create key.json --iam-account=quiz-account@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com
export GOOGLE_APPLICATION_CREDENTIALS=key.json

echo "Setting quiz-account IAM Role"
gcloud projects get-iam-policy $DEVSHELL_PROJECT_ID --format json > iam.json
node setup/add_iam_policy_to_service_account.js
gcloud projects set-iam-policy $DEVSHELL_PROJECT_ID iam_modified.json

echo "Project ID: $DEVSHELL_PROJECT_ID"
```
To start the worker application, enter the following command:
```sh
npm run worker
```
## Review the case study application
To view the application, click **Web preview > Preview on port 8080.**</br>
![d](https://user-images.githubusercontent.com/73010204/142590336-cbae670a-f98d-47a4-89d8-1383aa5bac35.PNG)</br>
Output : </br>
![e](https://user-images.githubusercontent.com/73010204/142590625-60bd5314-19e5-4d4e-a70b-a2051bb5ee8a.png)</br>
In the navigation bar, click **Take Test**.</br>
Click **Places.**</br>
Answer the question.After you answer the question, you should see a final screen inviting you to submit feedback.Quiz takers will be able to select a rating and enter feedback text.</br>
![2](https://user-images.githubusercontent.com/73010204/142828259-4378222e-89d0-4570-868d-dd5808e81653.png)
## Examining the Case Study Application Code
### Launch the Cloud Shell Editor
In Cloud Shell, click **Open Editor**. Click Open in a new window.</br>
### Review the GCP application code structure
Navigate to the **pubsub-languageapi-spanner/start** folder using the file browser panel on the left side of the editor.</br>
Select the **pubsub.js** file in the .../server/gcp folder.
- This file contains a module that allows applications to publish feedback messages to a Cloud Pub/Sub topic and register a callback to receive messages from a Cloud Pub/Sub subscription.

Select the **languageapi.js** file in the **...server/gcp** folder.
- This file contains a module that allows users to send text to the Cloud Natural Language ML API and to receive the sentiment score from the API.

Select the **spanner.js** file in the **...server/gcp** folder.
- This file contains a module that allows users to save the feedback and Natural Language API response data in a Cloud Spanner database instance.

![3](https://user-images.githubusercontent.com/73010204/142853154-fc58c3a5-4bdc-47c9-adb8-eb40a726dc0f.png)

### Review the web application code
Select the **index.js** file in the **...server/api** folder.
- The handler for POST messages sent to the _/api/quizzes/feedback/:quiz_ route publishes the feedback data received from the client to Pub/Sub.

Select the **worker.js** file in the **...console** folder.
- This file runs as a separate console application to consume the messages delivered to a Pub/Sub subscription.

## Working with Cloud Pub/Sub
### Create a Cloud Pub/Sub topic
- Navigate to the **Cloud Platform Console.**
- Select **Navigation menu > Pub/Sub.**
- Click **Create Topic**.
- For Topic ID, type _feedback_, and then click **CREATE TOPIC**.

![4](https://user-images.githubusercontent.com/73010204/142854237-b81c8fc3-e218-4b68-947b-540282a7917d.png)
### Create a Cloud Pub/Sub topic
Go to the second **Cloud Shell** window.</br>
To create a Cloud Pub/Sub subscription named _cloud-shell-subscription_ against the _feedback_ topic, execute the following command:
```sh
gcloud pubsub subscriptions create cloud-shell-subscription --topic feedback
```
### Publish a message to a Cloud Pub/Sub topic
To publish a "Hello World" message into the _feedback_ topic, execute the following command:
```sh
gcloud pubsub topics publish feedback --message "Hello World"
```
## Retrieve a message from a Cloud Pub/Sub subscription
To pull the message from the _feedback_ topic with automatic acknowledgement of the message, execute the following command:
```sh
gcloud pubsub subscriptions pull cloud-shell-subscription --auto-ack
```

## Publishing Messages to Cloud Pub/Sub Programmatically
In this section, you write code to publish messages to Cloud Pub/Sub.</br>
Simply, the flow is look like</br>
Users click **"Send Feedback" ---> api/index.js ---> gcp/pubsub.js**
### Import, use and publish a message to Cloud Pub/Sub module
Open the ***...gcp/pubsub.js*** file in the editor.
```sh
const config = require('../config');
// TODO: Load the Cloud Pub/Sub module
const {PubSub} = require('@google-cloud/pubsub');
// END TODO
// TODO: Create a client object against Cloud Pub/Sub
// The PubSub(...) factory function accepts an options
// object which is used to specify which project's Cloud
// Pub/Sub topics should be used via the projectId
// property.
// The projectId is retrieved from the config module.
// This module retrieves the project ID from the
// GCLOUD_PROJECT environment variable.
const pubsub = new PubSub({
  projectId: config.get('GCLOUD_PROJECT')
});
// END TODO
// TODO: Get a reference to the feedback topic
// This code assumes that the topic is already present in
// the project.
// If it isn't then you would need to handle that case by
// using the pubsub object's createTopic(...) method
const feedbackTopic = pubsub.topic('feedback');
// END TODO
```
```sh
  function publishFeedback(feedback) {
  // TODO: Publish a message to the feedback topic
  // This runs asynchronously so you need to return the
  // Promise that results from executing topic.publish(...)
  // The feedback object must be converted to a buffer
  // In addition, it's a good idea to use a consistent
  // property for the message body. This lab will use the
  // name dataBuffer for the message data
  const dataBuffer=Buffer.from(JSON.stringify(feedback))
  return feedbackTopic.publish(dataBuffer);
  // END TODO
}
```
Save the file
### Invoke the Pub/Sub publish functionality
- In the **.../api/index.js** file, load the _'../gcp/pubsub'_ module.
- In the _post(...)_ handler for the '/feedback/:quiz' route, invoke the _publisher.publishFeedback(feedback)_ metehod that we wrote code above
- Then, return a response to the client indicating that feedback was received.
- In the catch block for error handling, invoke the next Express middleware.

**api/index.js**
```sh
/**
 * POST /api/quizzes/feedback/:quiz
 *
 * Submit the quiz feedback, get a response
 */
  router.post('/feedback/:quiz', (req, res, next) => {
  const feedback = req.body; // in the form [{id, answer}]
  console.log(feedback);
  // TODO: Publish the message into Cloud Pub/Sub
  publisher.publishFeedback(feedback).then(() => {
    // TODO: Move the statement that returns a message to
    // the client app here
    res.json('Feedback received'); // moved here
    // END TODO
      // TODO: Add a catch
    }).catch(err => {
        // TODO: There was an error, invoke the next middleware
      next(err);
    // END TODO
    });
  // END TODO
});
```
Save the file

### Run the application and create a Pub/Sub message
- In the first Cloud Shell window, restart the web application (if it is running, stop and start it): _npm start_
- Preview the web application.
- Click **Take Test**.
- Click **Places**.
- Answer the question, select the rating, enter some feedback text, and click **Send Feedback**.
![5](https://user-images.githubusercontent.com/73010204/142879796-50e02f1c-3ed0-4fb6-aa15-727f3fb8d316.png)</br>
- In the second Cloud Shell window, to pull a message from the _cloud-shell-subscription_, execute the following command:
```sh
gcloud pubsub subscriptions pull cloud-shell-subscription --auto-ack
```
Output:
![6](https://user-images.githubusercontent.com/73010204/142879802-a58fa21c-6c44-4938-873b-375b259cda8e.png)</br>

## Subscribing to Cloud Pub/Sub Topics Programmatically
In this section you will write the code to create a subscription to a Cloud Pub/Sub topic and receive message notifications in the worker console application.</br>
Users click **"Send Feedback"**  ---_message_--->  **console/worker.js**   ---handler---> **gcp/pubsub.js**
### Write code to create a Cloud Pub/Sub subscription and receive messages
- Return to the _...gcp/pubsub.js_ file.
- In the _registerFeedbackNotification(cb)_ function, create a subscription called _'worker-subscription'_ against the feedback topic.
- Configure the subscription options to automatically acknowledge messages.
- Add an error handler to trap the error if the subscription already exists.
- Using the subscription object, use the get() method to return a promise to register an event handler for message events.
- In the promise also register an error handler for error events.
- Add a final catch to the promise to log errors.

**pubsub.js**
```sh
// The worker application will pass a callback to this
// method as the cb argument so it is notified when a
// feedback PubSub message is received
function registerFeedbackNotification(cb) {
  // TODO: Create a subscription called worker-subscription
  // TODO: Have it auto-acknowledge messages
  feedbackTopic.createSubscription('worker-subscription', { autoAck: true }, (err, feedbackSubscription) => {
    // TODO: Trap errors where the subscription already exists
    // Create a subscription object for worker-subscription if
    // the subscription already exists
    // err.code == 6 means subscription already exists
    if (err && err.code == 6) {
        // subscription already exists
        console.log("Feedback subscription already exists");
        feedbackSubscription=feedbackTopic.subscription('worker-subscription')
    }
    // END TODO
    // TODO: Use the get() method on the subscription object to call
    // the API request to return a promise
    feedbackSubscription.get().then(results => {
      // The results argument in the promise is an array - the
      // first element in this array is the subscription object.
      // TODO: Declare a subscription constant
      const subscription = results[0];
      // END TODO
      // TODO: Register an event handler for message events
      // Use an arrow function to handle the event
      // When a message arrives, invoke a callback
      subscription.on('message', message => {
          cb(message.data);
      });
      // END TODO
      // TODO: Register an event handler for error events
      // Print the error to the console
      subscription.on('error', err => {        
          console.error(err);
      });
      // END TODO
    })
    // END TODO for the get() method promise
    // TODO
    // Add a final catch to the promise to handle errors
    .catch(error => { console.log("Error getting feedback subscription", error)});
    // END TODO
  });
  // END TODO for the create subscription method
}
```
### Write code to use the Pub/Sub subscribe functionality
- In the _.../console/worker.js_ file, load the _'../server/gcp/pubsub'_ module.
- In the handler(message) function, and after the existing code, log the message to the console.

**console/worker.js**
```sh
// TODO: Load the ../server/gcp/pubsub module
const subscriber = require('../server/gcp/pubsub');
// END TODO
// TODO: Load the ../server/gcp/languageapi module
// END TODO
// TODO: Load the ../server/gcp/spanner module
// END TODO
console.log('Worker starting...');
// The callback function - invoked when a message arrives
function handler(message) {
   console.log('Message received');
  // TODO: Log the message to the console
  var messageData = JSON.parse(message.toString());
  console.log(messageData);
  // END TODO
```
At the end of the file, add the following code to register the handler
```sh
function as the Pub/Sub subscription callback.
// TODO: Register the callback with the module
subscriber.registerFeedbackNotification(handler);
// END TODO
```
Save the file.
### Run the web and worker application and create a Pub/Sub message
- In the first Cloud Shell window, start the web application if it's not already running:  _npm start_
- In the second Cloud Shell window, start the worker application: _npm run worker_
- Click **Take Test**.
- Click **Places.**
- Answer the question, select the rating, enter some feedback text, and then click **Send Feedback.**
- Return to the second Cloud Shell window. You should see that the worker application has received the feedback message via its handler and displayed it in the window
![7](https://user-images.githubusercontent.com/73010204/142882305-7d824751-5201-48ae-a33b-320700e1172b.png)
- Return to the second Cloud Shell window, and press **Ctrl+C** to stop the worker application.
- Return to the first Cloud Shell window, and press **Ctrl+C** to stop the web application.

## Using the Cloud Natural Language API
In this section you will write the code to perform sentiment analysis on the feedback text submitted by the user.</br>
Users click **“Send Feedback”**  -—message-—> **console/worker.js** --—handler(integrated Natural Language API functionality) -—> **gcp/pubsub.js**
### Write code to invoke the Cloud Natural Language API
- In the editor, move to the top of the **...gcp/languageapi.js** file.
- Load the **'@google-cloud/language** module.
- Create a new Language Service Client. This client provides access to Google Cloud Natural Language API text analysis operations such as sentiment analysis and entity recognition.
- Move to the _analyze(...) function_, and create an object literal to pass the text data for analysis as a parameter to the analysis methods of the Natural Language client object.
- Configure this parameter object with two properties: _content_ and _type_.
- Assign the feedback text to this object's _content_ property.
- Set the _type_ property value to _PLAIN_TEXT_
- Pass the parameter object to the Language Service Client's analyzeSentiment method to analyze the document.
    - The Cloud Natural Language API expects an object with the form:
{ document: { content: 'Content to be analyzed', type: 'SOME_TYPE' } }
- Then, return the sentiment score from the Natural Language API.

**languageapi.js**
```sh
// Import the config module
const config = require('../config');
// TODO: Load the Natural Language ML API module
const Language = require('@google-cloud/language');
// END TODO
// TODO: Create a client object against the Language API
// using the Language.LanguageServiceClient function
// The LanguageServiceClient function accepts an options
// object which is used to specify which project should be
// billed for use of the API via the projectId property.
// The projectId is retrieved from the config module.
// This module retrieves the project ID from the
// GCLOUD_PROJECT environment variable.
const language = new Language.LanguageServiceClient({
    projectId: config.get('GCLOUD_PROJECT')
});
// END TODO
function analyze(text) {
    // TODO: Create an object named document with the
    // correct structure for the Natural Language ML API
    // TODO: Initialize object content & type properties
    // TODO: Set content from text arg
    // TODO: Set type to PLAIN_TEXT
    const document = {
        content: text,
        type: 'PLAIN_TEXT'
    };
    // END TODO
    // TODO: Perform sentiment detection
    return language.analyzeSentiment({ document })
    // TODO: Chain then
    // When the results come back
    // The sentiment data is the first element
       .then(results => {
          const sentiment = results[0];
          // TODO: Get the sentiment score (-1 to +1)
          return sentiment.documentSentiment.score;
       });
    // END TODO
}
```
Save the file
### Write code to use the Natural Language API functionality
- In the **.../console/worker.js** file, load the _'../server/gcp/languageapi'_ module.
- In the _handler(message)_ function, and after the existing code, perform sentiment detection on the feedback.
- Then, log the score to the console.
- Assign a new score property to the feedback object.
- Return the message data.

**console/worker.js**
```sh
  // TODO: Load the ../server/gcp/languageapi module
const languageAPI = require('../server/gcp/languageapi');
// END TODO
console.log('Worker starting...');
// The callback function - invoked when a message arrives
function handler(message) {
  console.log('Message received');
  // TODO: Log the message to the console
  var messageData = JSON.parse(message.toString());
  console.log(messageData);
  // END TODO
  // TODO: Invoke the languageapi module method
  // with the feedback from the student
  languageAPI.analyze(messageData.feedback)
  .then(score => {
    // TODO: Log sentiment score
    console.log(`Score: ${score}`);
    // END TODO
    // TODO: Add a score property to feedback object
    // and return updated feedback object
    messageData.score = score;
    return messageData;
    // END TODO
  })
  // END TODO
  // TODO: Pass on the feedback object
  // to next Promise handler
  // END TODO
  // TODO: Add third .then(...)
    // TODO Log feedback saved message
    // END TODO
  // END TODO
  // TODO close off the promise chain with a catch() and log
  // any errors to the console
  // END TODO
}
```
Save the file.
### Run the web and worker application and test the Natural Language API
- Return to the first Cloud Shell window.
- Start the web application: _npm start_
- Switch to the second Cloud Shell window.
- Start the worker application: _npm worker start_
- Preview the web application.
- Click **Take Test**
- Click **Places**.
- Answer the questions, select the rating, enter some feedback text, and then click **Send Feedback**
![a](https://user-images.githubusercontent.com/73010204/142973940-3ad39a59-f6d4-4b43-97ce-ced4b487e9ac.png)
- Return to the second Cloud Shell window. You should see that the worker application has invoked the Cloud Natural Language API and displayed the **sentiment score** in the console.
![b](https://user-images.githubusercontent.com/73010204/142974007-6102a4d2-5cb5-481c-8985-36b2b07759f3.png)

## Persisting Data to Cloud Spanner
In this section you will create a Cloud Spanner instance, database, and table. Then you will write the code to persist the feedback data into the database.</br>
Users click **“Send Feedback”** -—message-—> **console/worker.js** --—handler(integrated Natural Language API functionality and Spanner) -—> **gcp/pubsub.js**


### Create a Cloud Spanner instance
- Return to the **Cloud Console**.
- On the **Navigation menu**, click **Spanner**
- Click **Create instance**.
- For **Instance name**, type _quiz-instance_
- In the **Choose a configuration** section, select **Regional**, and then select _us-central1_ as the region.
- Click **Create**.
![c](https://user-images.githubusercontent.com/73010204/142974859-9bbe8631-3eae-4fad-934d-9dc467967c9f.png)

### Create a Cloud Spanner database and table
- On the **Instance Details** page for _quiz-instance_, click **Create database**.
- For **Name**, type _quiz-database_.
- Under **Define your schema**, type the following SQL statement:
```sh
CREATE TABLE Feedback (
    feedbackId STRING(100) NOT NULL,
    email STRING(100),
    quiz STRING(20),
    feedback STRING(MAX),
    rating INT64,
    score FLOAT64,
    timestamp INT64 )
    PRIMARY KEY (feedbackId);
```
- Click **Create**
<.br>![d](https://user-images.githubusercontent.com/73010204/142975118-592b2c76-8093-49d8-ae7d-a39b2ec3d466.png)

### Write code to persist data into Cloud Spanner
- Return to the code editor, and move to the top of the **...gcp/spanner.js** file.
- Load the _'@google-cloud/spanner'_ module.
- Create a new Cloud Spanner client.
- Get a reference to the Spanner instance.
- Get a reference to the Spanner database.
- Get a reference to the feedback Spanner database table.

**spanner.js**
```sh
// Import the config module
const config = require('../config');
// TODO: Import the @google-cloud/spanner module
const {Spanner} = require('@google-cloud/spanner');
// END TODO
// TODO: Create a client object to access Cloud Spanner
// The Spanner(...) factory function accepts an options
// object which is used to select which project's Cloud
// Spanner database instance(s) should be used via the
// projectId property.
// The projectId is retrieved from the config module.
// This module retrieves the project ID from the
// GCLOUD_PROJECT environment variable.
const spanner = new Spanner({
    projectID: config.get('GCLOUD_PROJECT')
});
// END TODO
// TODO: Get a reference to the Cloud Spanner instance
const instance = spanner.instance('quiz-instance');
// END TODO
// TODO: Get a reference to the Cloud Spanner database
const database = instance.database('quiz-database');
// END TODO
// TODO: Get a reference to the Cloud Spanner table
const feedbackTable = database.table('feedback');
// END TODO
```
- Move to the _saveFeedback(...)_ function.
- Create a 'reversed' email. Example Original: _app.dev.student@example.org_ Reversed: _org_example_student_dev_app_
- Create a record object to insert into Spanner using the feedback object's properties. You will need to use _Spanner.float (score)_ to ensure that the Natural Language score has the correct format for Cloud Spanner.
- Create a key for this record from the 'reversed' email, quiz, and timestamp.
- Use await to insert the record into the feedback table inside a try block and catch any errors.
- Add a catch block and log a message for any errors. Spanner uses _err.code==6_ to indicate a record with the same id already exists, which indicates a duplicate message from PubSub in this case.

**spanner.js**
```sh
async function saveFeedback(
    { email, quiz, timestamp, rating, feedback, score }) {
    // TODO: Declare rev_email constant
    // TODO: Produce a 'reversed' email address
    // eg app.dev.student@example.org -> org_example_student_dev_app
    const rev_email = email
        .replace(/[@\.]/g, '_')
        .split('_')
        .reverse()
        .join('_');
    // END TODO
    // TODO: Create record object to be inserted into Spanner
    const record = {
        feedbackId: `${rev_email}_${quiz}_${timestamp}`,
        email,
        quiz,
        timestamp,
        rating,
        score: Spanner.float(score),
        feedback,
    };
    // END TODO
    // TODO: Insert the record into the table
    // use try {} catch {} and check for err.code==6 to trap
    // insert errors caused by duplicated PubSub messages
    try {
        console.log('Saving feedback');
        await feedbackTable.insert(record);
    } catch (err) {
        if (err.code === 6 ) {
            console.log("Duplicate message - feedback already saved");
        } else {
            console.error('ERROR processing feedback:', err);
        }
    }
    // END TODO
}
```
Save the file
### Write code to use the Cloud Spanner functionality
- In the **.../console/worker.js** file, load the _'../server/gcp/spanner'_ module.
- In the _handler(message)_ function, add a second _.then(...)_ chained method to the first one.
- In the body of the second _.then(...)_ method, pass a reference to the method that saves the feedback into Spanner.
- The second _.then(...)_ method also returns a Promise, so add a third _.then(...)_ chained method to the end.
- Use an arrow function as the callback in the _.then(...)_ method with no arguments and an empty body.
- In the body of the arrow function, log a message to the console to say that the feedback has been saved.
- Close off the promise chain with a catch and log any errors to the console.

**console/worker.js**
```sh
// TODO: Load the ../server/gcp/pubsub module
const subscriber = require('../server/gcp/pubsub');
// END TODO
// TODO: Load the ../server/gcp/languageapi module
const languageAPI = require('../server/gcp/languageapi');
// END TODO
// TODO: Load the ../server/gcp/spanner module
const feedbackStorage = require('../server/gcp/spanner');
// END TODO
// The callback function - invoked when a message arrives
function handler(message) {
  console.log('Message received');
  // TODO: Log the message to the console
  var messageData = JSON.parse(message.toString());
  console.log(messageData);
  // END TODO
  // TODO: Invoke the languageapi module method
  // with the feedback from the student
  languageAPI.analyze(messageData.feedback)
  .then(score => {
    // TODO: Log sentiment score
    console.log(`Score: ${score}`);
    // END TODO
    // TODO: Add a score property to feedback object
    messageData.score = score;
    return messageData;
  })
  // TODO: Pass on the feedback object
  // to next Promise handler
  .then(feedbackStorage.saveFeedback)
  // END TODO
  // TODO: Add third .then(...)
  .then(() => {
      // TODO Log feedback saved message
      console.log('Feedback saved');
      // END TODO
  })
  // END TODO
  // TODO close off the promise with a catch and log
  // any errors
  .catch(console.error);
  // END TODO
}
// TODO: Register the callback with the module
subscriber.registerFeedbackNotification(handler);
// END TODO
```
### Run the web and worker application and test Cloud Spanner
- Save all the files, and then return to the first Cloud Shell window.
- If the web application is not running, start it: _npm start_
- Switch to the second Cloud Shell window.
- Restart the worker application: _npm run worker_
- Preview the web application.
- Click **Take Test**.
- Click **Places**.
- Answer the questions, select the rating, enter some feedback text, and then click **Send Feedback**.
![e](https://user-images.githubusercontent.com/73010204/142976276-1574f66f-48e8-4834-bb4c-e2e23562dd1b.png)
- Return to the second Cloud Shell window.
- Return to the Cloud Platform Console.
- On the **Navigation menu**, click **Spanner**.
- Select **quiz-instance > quiz-database > Query**.
- To execute a query, for Query, type _SELECT * FROM Feedback_, and then click **Run**. You should see the new feedback record in the Cloud Spanner database, including the message data from Cloud Pub/Sub and the score from the Cloud Natural Language API
![f](https://user-images.githubusercontent.com/73010204/142976495-cde1bb52-068e-400b-a9d2-d2877f82b22b.PNG)

## Congratulations!
We did
- Create a Cloud Pub/Sub topic.
- Publish messages to the topic.
- Subscribe to the topic to receive messages in a separate worker application.
- Use the Cloud Natural Language Machine Learning API.
- Create a Cloud Spanner database instance.
- Develop a Cloud Spanner database schema.
- Insert data into a Cloud Spanner database.

## Reference
**Google Cloud Fundamentals: Securing and Integrating Components of your Application** course
https://www.coursera.org/learn/securing-integrating-components-app/home/welcome
