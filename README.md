# MyApp
https://blog.angulartraining.com/fake-your-angular-backend-until-you-make-it-8d145f713e14

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) ver
sion 13.3.6.

## Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The application will automatically reload if you change any of the source files.

## Code scaffolding

Run `ng generate component component-name` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module`.

## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via a platform of your choice. To use this command, you need to first add a package that implements end-to-end testing capabilities.

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI Overview and Command Reference](https://angular.io/cli) page.

Install JSON Server
JSON Server is available as a package that you can install with NPM:
npm install --save json-server

Create the database file
Create the api/db.json file with the following content:

{
  "teams": [
    {
      "id": 1,
      "name": "FC Barcelona",
      "coach": "Ernesto Valverde",
      "description": "The best football team in the world!"
    },
    {
      "id": 2,
      "name": "Real Madrid",
      "coach": "Zinedine Zidane",
      "description": "The worst football team in the world!"
    }
  ]
}

The file api/db.json defines our API endpoints. In this example, we have one endpoint called /teams that will return a list of football teams.

I am talking about the real football, not American football 😉.

Create a route mapping file
Create a file named api/routes.json with the following content:

{
  "/api/*": "/$1"
}
Add a script to start the server
Add the following key/value pair to the scripts section of your package.json file:

"api": "json-server api/db.json --routes api/routes.json --no-cors=true"
Start the mock API
Now, we are ready to start the mock API with the following command:

npm run api
This will launch our mock API on http://localhost:3000.

Set up the HttpClientModule
Before being able to send HTTP requests to a backend, you must set up HttpClientModule. Open the app.module.ts file and add HttpClientModule to the imports array of the AppModule:

import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
​
import { AppComponent } from './app.component';
import { HttpClientModule } from '@angular/common/http';
​
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
Make a request to the backend
You can now inject the HttpClient service in your root component—AppComponent. Next, create a property called teams$ that will hold the Observable of teams returned from our GET request.

import { Component } from '@angular/core';
import { HttpClient } from '@angular/common/http';
​
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  teams$ = this.http.get('http://localhost:3000/api/teams');
​
  constructor(private http: HttpClient) {}
}
Display the list of teams
We can now use the async pipe to and the *ngFor directive inside our component's template to display the list of teams:

<h2>Teams</h2>
<div *ngFor="let team of teams$ | async">
  <h3>{{team.name}}</h3>
  <p>{{team.description}}</p>
</div>
Launch the application
We are ready to serve the application:

ng serve --open
The will automatically open your browser thanks to the --open flag. But hey, nothing is displayed on the page. Open your browser's dev tools and you will see that the request failed due to CORS issues.


This happens because our Angular application and our backend are not on the same domain — different ports tantamount to different domain names.

Solving CORS issues
To solve CORS issues, the easiest way is to avoid cross-domain HTTP calls altogether. Most of the time, our frontend and our backend reside in the same domain so we don’t have any CORS issue in production. This is a development time issue and the solution is to take advantage of the Angular CLI’s development server's built-in proxy and make all our HTTP call relative.

Make our HTTP calls relative
Replace http://localhost:3000/api/teams with /api/teams inside the component:
Solving CORS issues
To solve CORS issues, the easiest way is to avoid cross-domain HTTP calls altogether. Most of the time, our frontend and our backend reside in the same domain so we don’t have any CORS issue in production. This is a development time issue and the solution is to take advantage of the Angular CLI’s development server's built-in proxy and make all our HTTP call relative.

Make our HTTP calls relative
Replace http://localhost:3000/api/teams with /api/teams inside the component:

teams$: Observable<any> = this.http.get('/api/teams');
In production, this request will hit http://my-app.com/api/teams if your frontend is deployed at my-app.com and your backend resides at my-app.com/api. But in development, the request will hit http://localhost:4200/api/teams while our backend resides at localhost:3000/api/teams. To solve this development time issue, let's set up a proxy using the Angular CLI.

Create the proxy configuration object
Create a file name proxy.conf.json at the root of your application:

{
  "/api": {
    "target": "http://localhost:3000"
  }
}
Open the angular.json file and update the serve target by adding the key/value pair "proxyConfig": "proxy.conf.json"

Update the workspace configuration file to use the proxy
{
  "serve": {
    "builder": "@angular-devkit/build-angular:dev-server",
    "options": {
      "browserTarget": "my-app:build",
      "proxyConfig": "proxy.conf.json"
    },
    "configurations": {
      "production": {
        "browserTarget": "my-app:build:production"
      }
    }
  }
}
With this configuration, in development mode, all requests that start with /api will be forwarded to http://localhost:3000 where our mock API resides. No CORS issues anymore and when we deploy our application to production, no change is required to our Angular application to make it work with a real backend.

Restart the server
Restart the server to take into account the proxy configuration and check your browser again.


Summary
In this article, we learned how to leverage JSON Server to build mock API. We also learned how to properly configure our Angular applications to take advantage of the CLI’s built-in proxy to avoid CORS issues in development mode.
