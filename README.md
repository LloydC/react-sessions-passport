# Authentication Examples

### Description

This project is an example of how you can perform authentication in React. This application has a login and a registration page that communicate to the application server through its API to authenticate users.  Users can register,login, log out and also sign in using an existing Google account.

It makes use of **Local Authentication**(a server-side authentication using Sessions) and **OAuth2 Authentication**(a client-side authentication using OAuth version 2 protocol to authenticate using Google).

Authentication is configured on both the client-side and server-side of this project:
- **client-side**:  OAuth2 authentication is facilitated by the use of `<GoogleLogin/>` Component provided by npm package [react-google-login](https://www.npmjs.com/package/react-google-login).
- **server-side**: Local Authentication and OAuth2 Authentication is handled by [Passport.js](http://www.passportjs.org/)

**Note:** This project handles only the main flow, alternative flows, like errors handling, are not covered and may present errors.

## Prerequisites

To run this project, you need to have:

- A running MongoDB instance (i.e. [MongoDB Atlas](https://www.mongodb.com/cloud/atlas))
- A Google Client ID and Client Secret, you get those by **creating an account and registering your application**  from  https://console.developers.google.com/  (You can follow this [guide](https://theonetechnologies.com/blog/post/how-to-get-google-app-client-id-and-client-secret) to create an account and a application).

  **Important:** Make sure to add `http://localhost:3000` for _Authorized redirect URIs_

## Installation

1. Clone this repository
2. Go to `server` folder and run:

```sh
npm i
```

3. Create a `.env` file with the following environment variables:

```sh
PORT=4000
ENV=development
MONGODB_URI='<MongoDB connection string>'
SERVER_API_URL=http://localhost:4000/api
GOOGLE_CLIENT_ID=<Client ID from your Google Developer Application>
GOOGLE_CLIENT_SECRET=<Secret from your Google Developer Application>
SECRET="<Random string used to encrypt message>"
```

4. Start the server:

```sh
npm start
```

5. Go to `client` folder and run:

```sh
npm i
```

6. Create a `.env` file with the following content:

```sh
REACT_APP_GOOGLE_CLIENT_ID=<Client ID from your Google Developer Application>
REACT_APP_SERVER_API_URL=http://localhost:4000/api
```

7. Start the client:

```sh
npm start
```

You should see the authentication page loaded in your browser.


## How Local Authentication Works

[Passport](http://www.passportjs.org/) module handles several types of authentication, the one used here is [Passport Local](http://www.passportjs.org/packages/passport-local/).

Strategy made for this authentication is defined in the `server/config/passport.js` file. 
Routing Authentication Handlers are defined in the `server/routes/authRoutes.js` file.

### Registration breakdown

On the client side, the `http://localhost/auth` route loads the registration form, the email and password are entered by a user then sent to the server through a POST request to the `/api/register` route.

The server receives the user information (`authRoutes.js` file) and checks if the email is already associated to another account, performing a search in MongoDB. If no user is found, a new one is created.

The user ID is then stored in a Session (the session is also stored in MongoDB).  

In our `app.js`, the functions `serializeUser` and `deserializeUser`(l36-46) are responsible for translating a user object to data to be stored in session (the ID information in this case) and vice-versa. The session ID is automatically sent to the client in the request's response.

When the client receives the server response, the session ID is automatically saved in a cookie and all the following requests will be sent with this information attached: this is how the server knows which session should be retrieved from MongoDB.

Finally the browser is redirected to a protected route(`/`). Protected routes should not be accessible without authentication, so these pages make requests to `/api/protected` route. This route validates the user, using `passport.authenticate` and returns if it is valid or not. The client should use this information to allow the page loading or not.

### Login breakdown

The Login page uses a similar pattern for performing the authentication of an existing user. 

On the client side, the user sends a POST request to the `/api/login` route with his/her login information, the email is received and used to search for a matching user in MongoDB.

If it is found, the password is compared with the one received from the request and the token generation is performed, like explained during registration.

## How Google Authentication Works

The React application is responsible for the communication with Google, possible due to the [React Google Login](https://www.npmjs.com/package/react-google-login) module, that provides A Google oAUth Sign-in / Log-in Component for React. In this case all the authentication process is performed by Google, the application trusts the logged user because it trusts the information provided by Google.

Strategy made for this authentication is defined in the `server/config/passport.js` file.

### Google Sign Breakdown

The `GoogleLogin` component, when clicked, opens a pop-up with the Google login page. After a successful authentication, Google redirects the browser back to the React application with an `accessToken` object. This object is then sent to a server route called '/api/login/google'.

Using `Passport` and [Google Token Strategy](https://www.npmjs.com/package/passport-google-token) the access token is validated through a new request to Google and a `profile` object with the user infomation (i.e: givenName, image, etc...) is returned. The next step is to check if there is already a user linked with that Google Account in MongoDB, this is done by trying to find a user with a `profileId` with the same value as the ID in `profile` object. If no user is found, a new one is created using the Google account information.

This is achieved by using two properties on the User model `providerId` and `provider`, the first one is used to **store the Google user ID** and its responsible to keep the link between the two accounts, `provider` is a string field that **stores the name of the authentication service**, _Google_ in this case, it is useful when several authentication providers are used (i.e: _Facebook_, _LinkedIn_, etc....) 

From now on, the process is the same as the one for Local Authentication.

[This article](https://medium.com/@alexanderleon/implement-social-authentication-with-react-restful-api-9b44f4714fa) has a complete example of using authentication with multiple services that also use OAuth.

## Other Topics

- This project was built using [Tailwind CSS](https://tailwindcss.com/) and prepared to use [Styled Components](https://styled-components.com/), if you want to configure yourself you can follow [this tutorial](https://www.youtube.com/watch?v=FiGmAI5e91M)
- This project also uses React Icons, you can see more of them [here](https://react-icons.github.io/react-icons/).
- If you would like to see another project that uses hooks you can check my Iron Beers project [here](https://github.com/lotofcaffeine/ironbeers-hooks).
- To know more about hooks or routing in React you can check the [Official Hooks Documentation](https://reactjs.org/docs/hooks-intro.html) and the [React Router Documentation](https://reacttraining.com/react-router/web/guides/quick-start).

<p align="center">
   <img src=".github/loop.JPG" width="200"/>
</p>
