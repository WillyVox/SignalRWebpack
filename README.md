## Tutorial: Get started with ASP.NET Core SignalR using TypeScript and Webpack
[https://learn.microsoft.com/en-gb/aspnet/core/tutorials/signalr-typescript-webpack?view=aspnetcore-8.0&tabs=visual-studio-code]
[https://github.com/dotnet/AspNetCore.Docs.Samples/tree/main/tutorials/signalr-typescript-webpack/samples/8.x/SignalRWebpack/Hubs]

Create the ASP.NET Core web app

    $ dotnet new web -o SignalRWebpack
    code -r SignalRWebpack

    $ dotnet add package Microsoft.TypeScript.MSBuild
    -> The preceding command adds the Microsoft.TypeScript.MSBuild package, enabling TypeScript compilation in the project.

### Configure the server
    In this section, you configure the ASP.NET Core web app to send and receive SignalR messages.

    1. In Program.cs, call AddSignalR: 
        var builder = WebApplication.CreateBuilder(args);
        builder.Services.AddSignalR();
    
    2. Again, in Program.cs, call UseDefaultFiles and UseStaticFiles:
        var app = builder.Build();
        app.UseDefaultFiles();
        app.UseStaticFiles();
    -> The preceding code allows the server to locate and serve the index.html file. The file is served whether the user enters its full URL or the root URL of the web app.
    
    3. Create a new directory named Hubs in the project root, SignalRWebpack/, for the SignalR hub class.
    
    4. Create a new file, Hubs/ChatHub.cs, with the following code:
        using Microsoft.AspNetCore.SignalR;

        namespace SignalRWebpack.Hubs;

        public class ChatHub : Hub
        {
            public async Task NewMessage(long username, string message) =>
                await Clients.All.SendAsync("messageReceived", username, message);
        }
    -> The preceding code broadcasts received messages to all connected users once the server receives them. 
    It's unnecessary to have a generic on method to receive all the messages. 
    A method named after the message name is enough.

    In this example:
        - The TypeScript client sends a message identified as newMessage.
        - The C# NewMessage method expects the data sent by the client.
        - A call is made to SendAsync on Clients.All.
        - The received messages are sent to all clients connected to the hub.

    5. Add the following using statement at the top of Program.cs to resolve the ChatHub reference:
        using SignalRWebpack.Hubs;

    6. In Program.cs, map the /hub route to the ChatHub hub. Replace the code that displays Hello World! with the following code:
        app.MapHub<ChatHub>("/hub");

### Configure the client
    In this section, you create a Node.js project to convert TypeScript to JavaScript and bundle client-side resources, including HTML and CSS, using Webpack.

    1. Run the following command in the project root to create a package.json file:
    $ npm init -y

    2. Add the highlighted property to the package.json file and save the file changes:
    -> Setting the private property to true prevents package installation warnings in the next step.

    3. Install the required npm packages. Run the following command from the project root:
    $ npm i -D -E clean-webpack-plugin css-loader html-webpack-plugin mini-css-extract-plugin ts-loader typescript webpack webpack-cli

    -> The -E option disables npm's default behavior of writing semantic versioning range operators to package.json. 
    For example, "webpack": "5.76.1" is used instead of "webpack": "^5.76.1". This option prevents unintended upgrades to newer package versions. 
    For more information, see the npm-install documentation.

    4. Replace the scripts property of package.json file with the following code:
    "scripts": {
        "build": "webpack --mode=development --watch",
        "release": "webpack --mode=production",
        "publish": "npm run release && dotnet publish -c Release"
    },

    -> The following scripts are defined:
    build: Bundles the client-side resources in development mode and watches for file changes. 
    The file watcher causes the bundle to regenerate each time a project file changes. 
    The mode option disables production optimizations, such as tree shaking and minification. use build in development only.
    release: Bundles the client-side resources in production mode.
    publish: Runs the release script to bundle the client-side resources in production mode. It calls the .NET CLI's publish command to publish the app.

    5. Create a file named webpack.config.js in the project root, with the following code:
    -> The preceding file configures the Webpack compilation process:
    The output property overrides the default value of dist. The bundle is instead emitted in the wwwroot directory.
    The resolve.extensions array includes .js to import the SignalR client JavaScript

    6. Create a new directory named src in the project root, SignalRWebpack/, for the client code.

    7. Copy the src directory and its contents from the sample project into the project root. The src directory contains the following files:
    index.html, which defines the homepage's boilerplate markup:

    8. Run the following command at the project root:
    $ npm i @microsoft/signalr @types/node
    -> The preceding command installs:
    The SignalR TypeScript client, which allows the client to send messages to the server.
    The TypeScript type definitions for Node.js, which enables compile-time checking of Node.js types.

### Test the app
    Confirm that the app works with the following steps:

    1. Run Webpack in release mode by executing the following command in the project root:
    $ npm run release
    This command generates the client-side assets to be served when running the app. The assets are placed in the wwwroot folder.

    Webpack completed the following tasks:
        Purged the contents of the wwwroot directory.
        Converted the TypeScript to JavaScript in a process known as transpilation.
        Mangled the generated JavaScript to reduce file size in a process known as minification.
        Copied the processed JavaScript, CSS, and HTML files from src to the wwwroot directory.
        Injected the following elements into the wwwroot/index.html file:
        A <link> tag, referencing the wwwroot/main.<hash>.css file. This tag is placed immediately before the closing </head> tag.
        A <script> tag, referencing the minified wwwroot/main.<hash>.js file. This tag is placed immediately after the closing </title> tag.
    
    2. Build and run the app by executing the following command in the project root:
    $ dotnet run
    -> The web server starts the app and makes it available on localhost.

    3. Open a browser to https://localhost:<port>. The wwwroot/index.html file is served. Copy the URL from the address bar.

    4. Open another browser instance (any browser). Paste the URL in the address bar.

    5. Choose either browser, type something in the Message text box, and select the Send button. The unique user name and message are displayed on both pages instantly.