# Diarize
Diarize is my final project submission for the CS50w course. A positive psychology focussed diary application that encourages users to plan and review their days to make the best possible use of their time. The format of the application is based on the popular <a href="https://createurbestself.com">6 minute diary</a> with a few tweaks and extras added.

## Distinctiveness and Complexity
### Django
Diarize has been built using Django, a high-level web framework that allows developers to quickly and iteratively build, prototype and deploy web applications. It does this by including a large amount of the common functionality of web applications (i.e. user account management, security, database management) right out of the box. The framework also encourages developers to work within this philosophy - the re-usable elements of a project (such as a comments system, or an basket order system) are housed in self-contained ‘app’ directories, so they can easily be picked up and slotted into other projects that require the same functionality. 

The conventions of the django development philosophy are enforced by the framework’s structure. The overall product is referred to as a ‘project’ and begins its life as follows:

```
manage.py
<project name>/
	__init__.py
	settings.py
	urls.py
	wsgi.py
```

The manage.py file contains the nuts and bolts that function as the command line API. The project directory contains the global settings and information that are used to stitch your project together. Functionality is then added to your project by creating new (or adding pre-prepared) apps. These are self contained directories that are added to the root, and look like so:

```
new_app/
	__init__.py
	admin.py
	apps.py
	migrations/
		__init__.py
	models.py
	tests.py
	views.py
```

Each of the files teases out the common requirements of a web-server - models.py contains python classes that are used to build and maintain a database (if required), views.py contains the code describing the functionality of the app, tests.py contains (unsurprisingly) test scripts and so on. Other files can be added to this basic structure as required, such as a urls.py file to tell django what view function to call when a url is requested, static HTML / CSS / front-end .js files can be added and so on.

Django was introduced to the CS50w course during the third project. So, for those interested, a far more comprehensive breakdown of the framework can be found in the readme for that site - a thrilling dive into the world of takeaway pizza.

## Project Structure and Logic

The Diarize project is split into two apps - `accounts/` and `diary/`:

**`accounts/`** handles user registration, login, authentication and logout. This app serves as a perfect example of the reusability that Django encourages, as it is lifted almost unchanged from project 3 mentioned above. A more detailed description of it's funcionality can be found in the readme for project 3 (also mentioned above).

**`diary/`** handles all of the actual functionality of diarize. The user journey through the app (once logged in) can be broken down quite nicely via the view functions in `views.py`:

* **`overview`**: once logged in, users first see the overview screen. This includes a diary page for each of the previous entries they have created - selecting any of these will take them to the relevant entry page. A blank entry appears in the top right of the page that allows users to create a new entry or, if they have already completed the plan stage of a new entry, allows users to complete an unfinished entry.
* **`plan_entry`**: once a user chooses to begin a new entry they will be taken to the plan page. Users are asked to complete each of the sections of the plan one by one and, once submitted, their entries are added to the app’s database. The page then links back to the overview screen which will now include the option to complete this new entry when they are ready
* **`review_entry`**: Once the day is over, users can review their day to complete an entry. Users are asked to review the sections of the plan they made earlier, and this information is recorded on submission to create a complete entry. The page redirects to the overview screen where the new entry will appear alongside the others.
* **`view_entry`**: A user can select any previous entry from the overview screen to see all of the information they entered when creating and completing the entry. 
* **`intro`**: when a user first creates an account, or requests a review of the site functionality, they are directed to the intro page. This guides the user step-by-step through each section of the application, with an explanation of the functionality and how it contributes to the user’s positive psychology

The back-end script in `views.py` tells only a small part of the story. A large portion of the site's interactive features is handled front-end via several small HTML files stitched together with client-side javascript. Each of the intro, plan and review routes has a similar structure and logic:
* The route’s view function in `views.py` (e.g `plan_entry`) loads a base template with an empty section for the page content
* The base template includes a javascript file that handles the rest of the page logic - these javascript files can be found in static/js `e.g plan.js`
* The javascript requests the page content as it is required via asynchronous (or AJAX) requests, inserts the content into the main section of the page, and adds the required interactive functionality to the HTML elements once loaded
* The page's html content is served by a specific `<route name>_pages` view in views.py - i.e. `plan_pages` loads the different page content for the plan route, the same for `intro_pages` and `review_pages`
* The different page content can be found in `static/html` directories of the same name as the route in views.py e.g. `static/html/plan_pages/`
* Once the user has completed their entry, the page javascript collects the user input and submits it to the respective route’s view function via a POST request

The remaining files in the app directories that aren't specifically mentioned are simply part of the wallpaper of the django framework, and are best described in the django documentation. There are some other features of the site, however, that warrant a mention:

## Bootstrap

To easily facilitate responsive content (and to save a huge amount of time doing so!) the app uses Bootstrap CSS. Bootstrap uses a grid layout that varies dependent on the width of the client viewport. So, for example, the page contents can be displayed in three columns for large screens, two for medium and one for small. The columns then simply stack on top of eachother when the viewport varies from large to small. This layout is specified using Bootstraps cornucopia of CSS class names, which can be seen all over the html files e.g. `col-md-3` specifies a column that takes up a quarter of the page (the page is divided into 12 - so 3 / 12) when the viewport is medium sized or larger.

Bootstrap also offers a huge range of custom components and utilities that can be used to add a professional touch to a site very quickly. These have been used more sparingly, with custom css (found in `static/css/`) being the preference given the site's more unique theme and look. 

## `load exmples.py`

To get things up-and-running more quickly, and to make testing easier, `load_example_data.py` includes instructions to load some example data into the app's database. The file contains a simple script that can be run using the django api's shell, creating a superuser account and quickly adding a few example entries under this account. This just makes life a little easier when making changes to the site, as it provides some example info to be displayed immediately.

## Dockerfile

Docker is a great way to ensure that a server / app / program etc. can be run easily and quickly on almost any hardware. As a service, docker uses vertualization to 'containerize' a tiny virtual machine in which your application can run. Starting at OS level, you specify only what your project requires to run, and then docker creates a container from which your server / app / program can then be used. It eliminates the age old issues of versioning and compatibility, where you have to ensure that you have the correct OS config / version of python / versions of each of the package requirements / the rest of the kitchen sink organised, in order to get your app running correctly.


Entire books can (and have been) written on the subject of Docker, but a good overview can be seen from the instructions contained within`Dockerfile`. Starting with a tiny OS, the file tells docker:

* we need python 3
* install the python packages in `requirements.txt`
* copy the files from our app directory
* run the django migration instructions to create the app database
* run the load_examples.py script to add some example info to the database
* run a development server

With this, any system with docker installed can run a development sever of the diarize app. Functionality varies between different systems, but in a linux environment one need only type:
 
 ```
$ docker build -t diarize .
$ docker run -it -p 8000:0000 diarize
 ```
 in a bash terminal, and Docker will do the rest. Simple as that. No need to check any other requirements or wrestle with any compatibility issues.
