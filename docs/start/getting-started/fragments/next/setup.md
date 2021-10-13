
## Create a new Next.js App

To set up the project, we'll first create a new Next.js app with [Create Next App](https://nextjs.org/docs/api-reference/create-next-app), a simple CLI tool that enables you to quickly start building a new Next.js application, with everything set up for you. We'll then add Amplify and initialize a new project.

From your projects directory, run the following commands:

```bash
npx create-next-app next-amplified
cd next-amplified
```

This creates a new Next.js app in a directory called `next-amplified` and then switches us into that new directory.

Now that we're in the root of the project, we can run the app by using the following command:

```bash
npm run dev
```

This runs a development server and allows us to see the output generated by the build, you can see the running app by navigating to `http://localhost:3000`.

## Initialize a new backend

Now that we have a running Next.js app, it's time to set up Amplify so that we can create the necessary backend services needed to support the app.

From the root of the project, run:

```bash
amplify init
```

When you initialize Amplify you'll be prompted for some information about the app:

```console
Enter a name for the project (nextamplified)

# All AWS services you provision for your app are grouped into an "environment"
# A common naming convention is dev, staging, and production
Enter a name for the environment (dev)

# Sometimes the CLI will prompt you to edit a file, it will use this editor to open those files.
Choose your default editor

# Amplify supports JavaScript (Web & React Native), iOS, and Android apps
Choose the type of app that you're building (javascript)

What JavaScript framework are you using (react)

Source directory path (src)

Distribution directory path (build)

Build command (npm run build)

Start command (npm start)

# This is the profile you created with the `amplify configure` command in the introduction step.
Do you want to use an AWS profile?
```

> Where possible the CLI will infer the proper configuration based on the type of project Amplify is being initialized in. In this case it knew we are using Create Next App and provided the proper configuration for type of app, framework, source, distribution, build, and start options.

When you initialize a new Amplify project, a few things happen:

- It creates a top level directory called `amplify` that stores your backend definition. During the tutorial you'll add capabilities such as a GraphQL API and authentication. As you add features, the `amplify` folder will grow with infrastructure-as-code templates that define your backend stack. Infrastructure-as-code is a best practice way to create a replicable backend stack.
- It creates a file called `aws-exports.js` in the `src` directory that holds all the configuration for the services you create with Amplify. This is how the Amplify client is able to get the necessary information about your backend services.
- It modifies the `.gitignore` file, adding some generated files to the ignore list
- A cloud project is created for you in the AWS Amplify Console that can be accessed by running `amplify console`. The Console provides a list of backend environments, deep links to provisioned resources per Amplify category, status of recent deployments, and instructions on how to promote, clone, pull, and delete backend resources

As you add or remove categories and make updates to your backend configuration using the Amplify CLI, the configuration in __aws-exports.js__ will update automatically.

## Install Amplify libraries

The first step to using Amplify in the client is to install the necessary dependencies:

```bash
npm install aws-amplify @aws-amplify/ui-react
```

The `aws-amplify` package is the main library for working with Amplify in your apps. The `@aws-amplify/ui-react` package includes React-specific UI components we'll use as we build the app.

Now that our Next.js app is set up and Amplify is initialized, we're ready to add an API in the next step.