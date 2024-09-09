step1:
For creating a mono Microfrontend app
ng new mono-workspace --create-application-false

step2:
Enter mono-workspace folder
cd mono-workspace

step3:

create application host-app with routing and styles

ng g application host-app --routing -style=scss

step4:

create application host-app with routing and styles

ng g application mfe-app --routing -style=scss

step5:
 We need module federation to run micrFrontend

 ng add @angular-architects/module-federation --project host-app --4200 
 ng add @angular-architects/module-federation --project host-app --4200

 step6:
 check if you can run the application
 ng s host-app 
 ng s mef-app 

 step7:

 work with host-app routing and add the routes to components which can be created specifically
 
 You can create specifically using 
 ng g c home project=host-app
  ng g c home project=mfe-app

  create to-dolist module and make the changes in host-app routing to load the the to-do list module on demand

   { path: 'todo-list', loadChildren: ()=>{
    return loadRemoteModule({
      remoteEntry: MFE_APP_URL,
      remoteName:"mfeApp",
      exposedModule: "./TodoListModule"
    }).then(m => m.TodoModule).catch(err => console.log(err));
  }}

  And in the webpack confic need to make below change so the app should run t http://localhost:4300/remoteEntry.js

  const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
const mf = require("@angular-architects/module-federation/webpack");
const path = require("path");
const share = mf.share;

const sharedMappings = new mf.SharedMappings();
sharedMappings.register(
  path.join(__dirname, '../../tsconfig.json'),
  [/* mapped paths to share */]);

module.exports = {
  output: {
    uniqueName: "mfeApp",
    publicPath: "auto",
    scriptType:"text/javascript"
  },
  optimization: {
    runtimeChunk: false
  },
  resolve: {
    alias: {
      ...sharedMappings.getAliases(),
    }
  },
  experiments: {
    outputModule: true
  },
  plugins: [
    new ModuleFederationPlugin({

        // For remotes (please adjust)
        name: "mfeApp",
        filename: "remoteEntry.js",
        exposes: {
            './TodoListModule': './projects/mfe-app/src/app/todo-list/todo-list.module.ts',
        },

        // For hosts (please adjust)
        remotes: {
            "mfeApp": "mfeApp@http://localhost:4200/remoteEntry.js",

        },


