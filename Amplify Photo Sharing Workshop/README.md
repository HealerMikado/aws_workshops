To create react application

npx create-react-app postagram

Now change into the new app directory and install NPM packages for AWS Amplify, AWS Amplify UI React, react-router-dom, emotion, and uuid

cd postagram
npm install aws-amplify @emotion/css uuid react-router-dom@5 @aws-amplify/ui-react


Configure Amplify
Installing the CLI

Next, we'll install the AWS Amplify CLI:

1
$ npm install -g @aws-amplify/cli

Now we need to configure the CLI with our credentials.
If you'd like to see a video walkthrough of this configuration process, click here 
.

$ amplify configure

- Specify the AWS Region: us-east-1 || us-west-2 || eu-central-1
- Specify the username of the new IAM user: amplify-cli-user
> In the AWS Console, click Next: Permissions, Next: Tags, Next: Review, & Create User to create the new IAM user. Then return to the command line & press Enter.
- Enter the access key of the newly created user:   
? accessKeyId: (<YOUR_ACCESS_KEY_ID>)  
? secretAccessKey: (<YOUR_SECRET_ACCESS_KEY>)
- Profile Name: amplify-cli-user

Initializing A New Project

$ amplify init

? Enter a name for the project: postagram

> The following configuration will be applied: 
Project information
| Name: postagram
| Environment: dev
| Default editor: Visual Studio Code
| App type: javascript
| Javascript framework: react
| Source Directory Path: src
| Distribution Directory Path: build
| Build Command: npm run-script build
| Start Command: npm run-script start

? Initialize the project with the above configuration? (Y/n): Y

Using default provider  awscloudformation
? Select the authentication method you want to use: AWS access keys
? accessKeyId:  (<YOUR_ACCESS_KEY_ID>)
? secretAccessKey:  (<YOUR_SECRET_ACCESS_KEY>)
? region:  (<REGION>)
Adding backend environment dev to AWS Amplify app: xxxxxxxxxxxxx

The Amplify CLI has initialized a new project, and you will see a new folder: amplify, as well as a new file called aws-exports.js in the src directory. These files contain your project configuration.

To view the status of the amplify project at any time, you can run the Amplify status command:

$ amplify status

    Current Environment: dev
    
┌──────────┬───────────────┬───────────┬─────────────────┐
│ Category │ Resource name │ Operation │ Provider plugin │
└──────────┴───────────────┴───────────┴─────────────────┘

To launch a new browser window and view the Amplify project in the Amplify console at any time, run the console command:

$ amplify console

Adding a GraphQL API

To add a GraphQL API, we can use the following command:

$ amplify add api

? Select from one of the below mentioned services: (Use arrow keys)
❯ GraphQL 
  REST 

? Here is the GraphQL API that we will create. Select a setting to edit or continue (Use arrow keys)
  Name: postagram 
  Authorization modes: API key (default, expiration time: 7 days from now) 
  Conflict detection (required for DataStore): Disabled 
  
? Continue

? Choose a schema template: (Use arrow keys)
❯ Single object with fields (e.g., “Todo” with ID, name, description) 
  One-to-many relationship (e.g., “Blogs” with “Posts” and “Comments”) 
  Blank Schema 

? Do you want to edit the schema now? (Y/n): Y 

The CLI should open this GraphQL schema in your text editor or IDE. If it doesnt, click on the link provided in the console to see this file

amplify/backend/api/postagram/schema.graphql

Update the schema to the following:

type Post @model {
  id: ID!
  name: String!
  location: String!
  description: String!
  image: String
}

After saving the schema, go back to the CLI and press enter


Deploying the API

To deploy the API, run the push command:


$ amplify push

# Amplify will fetch updates to the backend from the cloud and compile your schema from the file you just edited.

    Current Environment: dev
    
┌──────────┬───────────────┬───────────┬───────────────────┐
│ Category │ Resource name │ Operation │ Provider plugin   │
├──────────┼───────────────┼───────────┼───────────────────┤
│ Api      │ postagram     │ Create    │ awscloudformation │
└──────────┴───────────────┴───────────┴───────────────────┘

? Are you sure you want to continue? (Y/n): Y
? Do you want to generate code for your newly created GraphQL API (Y/n): Y
? Choose the code generation language target (Use arrow keys)
❯ javascript 
  typescript 
  flow 
? Enter the file name pattern of graphql queries, mutations and subscriptions (src/graphql/**/*.js): src/graphql/**/*.js
? Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions (Y/n): Y
? Enter maximum statement depth [increase from default if your schema is deeply nested] (2): 2 

Alternately, you can run $ amplify push -y to answer Yes to all questions.

Now the API is live and you can start interacting with it!
Testing the API

To test it out we can use the GraphiQL editor in the AppSync dashboard. To open the AppSync dashboard, run the following command:


$ amplify console api
? Select from one of the below mentioned services: (Use arrow keys)
❯ GraphQL 
  REST 

In the AppSync dashboard, click on Queries to open the GraphiQL editor. In the editor, create a new post with the following mutation:


mutation createPost {
  createPost(input: {
    name: "My first post"
    location: "New York"
    description: "Best burgers in NYC - Jackson Hole"
  }) {
    id
    name
    location
    description
  }
}

Then, query for the posts:
query listPosts {
  listPosts {
    items {
      id
      name
      location
      description
    }
  }
}

Using GraphQL with React

Now, our API is created & we can test it out in our app!

The first thing we need to do is to configure our React application to be aware of our Amplify project. We can do this by referencing the auto-generated aws-exports.js file that is now in our src folder.

To configure the app, open src/index.js and add the following code below the last import:

1
2
3
import Amplify from 'aws-amplify'
import config from './aws-exports'
Amplify.configure(config)

Now, our app is ready to start using our AWS services.
Interacting with the GraphQL API from our client application - Querying for data

Now that the GraphQL API is running we can begin interacting with it. The first thing we'll do is perform a query to fetch data from our API.

To do so, we need to:

    define the query
    execute the query
    store the returned data in our app state
    list the items in our UI

The main thing to notice in this component is the API call. Take a look at this piece of code:

1
2
/* Call API.graphql, passing in the query that we'd like to execute. */
const postData = await API.graphql({ query: listPosts });

src/App.js

Update your src/App.js file with the following code, which incorporates the snippet above - calling the GraphQL API

// src/App.js
import React, { useState, useEffect } from 'react';

// import API from Amplify library
import { API } from 'aws-amplify'

// import query definition
import { listPosts } from './graphql/queries'

export default function App() {
  const [posts, setPosts] = useState([])
  useEffect(() => {
    fetchPosts();
  }, []);
  async function fetchPosts() {
    try {
      const postData = await API.graphql({ query: listPosts });
      setPosts(postData.data.listPosts.items)
    } catch (err) {
      console.log({ err })
    }
  }
  return (
    <div>
      <h1>Hello World</h1>
      {
        posts.map(post => (
          <div key={post.id}>
            <h3>{post.name}</h3>
            <p>{post.location}</p>
          </div>
        ))
      }
    </div>
  )
}

In the above code we are using API.graphql to call the GraphQL API, and then taking the result from that API call and storing the data in our state. This should be the list of posts you created via the GraphiQL editor.

Next, test the app - in the terminal type:

1
$ npm start

make sure you are in folder ~/environment/postagram

Authentication

In this section we'll add authentication to our app, using AWS Cognito and the provided NPM module from @aws-amplify/ui-react

Adding Authentication

Next, let's update the app to add authentication.

To add the authentication service, we can use the following command:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
$ amplify add auth

Using service: Cognito, provided by: awscloudformation
 
 The current configured provider is Amazon Cognito. 
 
? Do you want to use the default authentication and security configuration? (Use arrow keys)
❯ Default configuration 
  Default configuration with Social Provider (Federation) 
  Manual configuration 
  I want to learn more.

?  How do you want users to be able to sign in? (Use arrow keys)
❯ Username 
  Email 
  Phone Number 
  Email or Phone Number 
  I want to learn more. 

?  Do you want to configure advanced settings? (Use arrow keys)
❯ No, I am done. 
  Yes, I want to make some additional changes. 

To deploy the authentication service, you can run the push command:

1
2
3
4
5
6
7
8
9
10
11
12
13
$ amplify push

    Current Environment: dev
    
┌──────────┬───────────────────┬───────────┬───────────────────┐
│ Category │ Resource name     │ Operation │ Provider plugin   │
├──────────┼───────────────────┼───────────┼───────────────────┤
│ Auth     │ postagram8917ba15 │ Create    │ awscloudformation │
├──────────┼───────────────────┼───────────┼───────────────────┤
│ Api      │ postagram         │ No Change │ awscloudformation │
└──────────┴───────────────────┴───────────┴───────────────────┘

? Are you sure you want to continue? Yes

When this step completes you will have authentication services set up in Amazon Cognito. To see more information, you can run the console command:

$ amplify console auth

Using service: Cognito, provided by: awscloudformation
? Which console
❯ User Pool
  Identity Pool
  Both


Authentication in React
Using the withAuthenticator component

To add authentication in the React app, we'll go into src/App.js and first import the withAuthenticator HOC (Higher Order Component) from @aws-amplify/ui-react:

1
2
3
// src/App.js, import the withAuthenticator component and associated CSS 
import { withAuthenticator } from '@aws-amplify/ui-react'
import '@aws-amplify/ui-react/styles.css';

Next, we'll wrap our default export (the App component) with the withAuthenticator HOC:

1
2
3
4
function App() {/* existing code here, no changes */}

/* src/App.js, change the default export to this: */
export default withAuthenticator(App)

Next test it out in the browser:

1
$ npm start

Now we can run the app and see that an Authentication flow has been added in front of our App component. This flow gives users the ability to sign up and sign in.

Click "Sign Up" and follow the prompts to create an account. Be sure to use a real email address! Once you submit your user information, check your email for a confirmation email to complete the sign up.

Now that you have the authentication service created, you can view it any time in the console by running the following command - select User Pool:

1
2
3
4
5
6
7
$ amplify console auth

Using service: Cognito, provided by: awscloudformation
? Which console
❯ User Pool
  Identity Pool
  Both

Adding a sign out button

You can also easily add a preconfigured UI component for signing out. First, modify the App function signature.

1
2
function App({ signOut, user }) {
  ...

Then to add the button, add the following at the end of the <div> in the returned JSX in src/App.js.

1
2
3
4
5
  <div>
    <h1>Hello World</h1>
  ...
    <button onClick={signOut}>Sign out</button>
  </div>

Styling the UI components

Next, let's update the UI component styling by setting styles for the :root pseudoclass.

To do so, open src/index.css and add the following styling:

1
2
3
4
5
:root {
  --amplify-primary-color: #006eff;
  --amplify-primary-tint: #005ed9;
  --amplify-primary-shade: #005ed9;
}

To learn more about theming the Amplify React UI components, check out the documentation here 
Accessing User Data

We can access the user's info now that they are signed in by calling Auth.currentAuthenticatedUser() in useEffect. Add the following code to src/App.js in the appropriate places

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
//Add an import to use the API and Auth components from aws-amplify
import { API, Auth } from 'aws-amplify'

//modify the useEffect() function to include the checkUser() function call
useEffect(() => {
  fetchPosts();
  checkUser(); // new function call
});

//define the checkUser function after the existing fetchPosts() function
async function checkUser() {
  const user = await Auth.currentAuthenticatedUser();
  console.log('user:', user);
  console.log('user attributes: ', user.attributes);
}

Image Storage with Amazon S3

To add image storage, we'll use Amazon S3, which can be configured and created via the Amplify CLI:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
$ amplify add storage

? Select from one of the below mentioned services: (Use arrow keys)
❯ Content (Images, audio, video, etc.) 
  NoSQL Database 

? Provide a friendly name for your resource that will be used to label this category in the project: › images

? Provide bucket name: › postagramxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx <use_the_default>  

? Who should have access: …  (Use arrow keys or type to filter)
  Auth users only
❯ Auth and guest users

? What kind of access do you want for Authenticated users? …  (Use arrow keys or type to filter)
 ✔ create/update
 ✔ read
 ✔ delete
(Use <space> to select, <ctrl + a> to toggle all)

? What kind of access do you want for Guest users? …  (Use arrow keys or type to filter)
  create/update
✔ read
  delete
(Use <space> to select, <ctrl + a> to toggle all)

? Do you want to add a Lambda Trigger for your S3 Bucket? N

To deploy the service, run the following command:

1
$ amplify push

To save items to S3, we use the Storage API. The Storage API works like this.
Saving an item

1
2
const file = e.target.files[0];
await Storage.put(file.name, file);

Retrieving an item

1
const image = await Storage.get('my-image-key.jpg')

Now we can start saving images to S3 and we can continue building the Photo Sharing Travel app. 


App Setup
Creating the folder structure for our app

Next, create the following files in the src directory:

    Button.js
    CreatePost.js
    Header.js
    Post.js
    Posts.js

1
2

$ touch src/Button.js src/CreatePost.js src/Header.js src/Post.js src/Posts.js

Next, we'll go one by one and update these files with our new code.
Button.js

Here, we will create a button that we'll be reusing across the app:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
import React from 'react';
import { css } from '@emotion/css';

export default function Button({
  title, onClick, type = "action"
}) {
  return (
    <button className={buttonStyle(type)} onClick={onClick}>
      { title }
    </button>
  )
}

const buttonStyle = type => css`
  background-color: ${type === "action" ? "black" : "red"};
  height: 40px;
  width: 160px;
  font-weight: 600;
  font-size: 16px;
  color: white;
  outline: none;
  border: none;
  margin-top: 5px;
  cursor: pointer;
  \:hover {
    background-color: #363636;
  }
`

Header.js

Add the following code in Header.js

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
import React from 'react';
import { css } from '@emotion/css';
import { Link } from 'react-router-dom';

export default function Header() {
  return (
    <div className={headerContainer}>
      <h1 className={headerStyle}>Postagram</h1>
      <Link to="/" className={linkStyle}>All Posts</Link>
    </div>
  )
}

const headerContainer = css`
  padding-top: 20px;
`

const headerStyle = css`
  font-size: 40px;
  margin-top: 0px;
`

const linkStyle = css`
  color: black;
  font-weight: bold;
  text-decoration: none;
  margin-right: 10px;
  \:hover {
    color: #058aff;
  }
`