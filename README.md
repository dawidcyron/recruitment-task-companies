# Companies earnings app

Solution for recruitment task. The goal of the app is to fetch data from API and display it in the form of table.

## Prerequisites

The project was created with Vue.js
* Node.js
* Yarn

## Installation
The project can be browsed in 2 ways. You can either launch the app in development mode with live server, or compile it to generate static files. The instruction will help you run the application in development mode. Start by navigating to the directory where you have cloned or downloaded the project, and run the commmand:
```bash
yarn install
```
This will install all of the necessary dependencies used by the project. To launch the development server and view the application running, run the command:
```bash
yarn serve
```
This will prepare all the files and launch a local server, available under ```http://localhost:8080```

## Usage
Once you open the app in your browser, you should see a spinner signaling the fetching of data from the API. After a few seconds, you should see the list of companies, limited to 10 per page. You you would like to change the amount of companies displayed in the table, you can edit the ```perPage``` property in ```src/components/DataTable.vue``` file.
## License
[MIT](https://choosealicense.com/licenses/mit/)