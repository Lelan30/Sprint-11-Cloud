# Sprint-11-Cloud
Sprint Challenge

Air Quality in the Cloud
Introduction
In this sprint challenge you will build a Flask-powered web application that displays data about air quality. You may use any tools and references you wish, but your final code should reflect your work and be saved in .py files (not notebooks).

You may use any imports/environments/tools you wish, but the baseline is:

flask, flask-sqlalchemy to build the application and data model
requests (as a dependency for API access)
It is also optional to manage your environment (e.g. pipenv) - it's highly suggested if available, but pipenv install flask flask-sqlalchemy requests is adequate for passing.

Part 1 - If I could put Flask in a File
This week we worked on a larger Flask project, but simple applications can fit entirely in a single file. Below is some starter code for a Flask web application. Please create a new file called aq_dashboard.py and add the following lines to it:

"""OpenAQ Air Quality Dashboard with Flask."""
from flask import Flask 

app = Flask(__name__) 

@app.route('/')
def root():
    """Base view."""
    return'TODO - part 2 and beyond!'
Ensure you are in a Python environment that at least has flask, flask-sqlalchemy, and requests. You can make a new isolated environment with pipenv install flask flask-sqlalchemy requests.

Finally, run the application with: FLASK_APP=aq_dashboard.py flask run

You should see something like:

* Serving Flask app "aq_dashboard.py" 
* Environment: production WARNING: Do not use the development server in a production environment. 
Use a production WSGI server instead. 
* Debug mode: off 
* Running on http://127.0.0.1:5000/ 
(Press CTRL+C to quit)
Visit the given URL in your browser to verify that it works.

It's not required, but you can enable debug mode if you find that it's helpful to you: FLASK_ENV='development' FLASK_DEBUG=1

Part 2 - Breathe Easy with OpenAQ
You'll need the openaq.py file to help you communicate with the API –and complete Part 2. Download You'll need the openaq.py file to help you communicate with the API –and complete Part 2. 

Our application is going to be a dashboard that displays air quality data from the Open AQ API Links to an external site.. This API does not require authorization and can be accessed directly via HTTP requests. 

Your first goal is to verify that you can pull data from OpenAQ with Python. You have been provided with a file –openaq.py– that provides some functions to help you query the API. These functions make use of the requests package. You'll still need to pip or pipenv install requests (if it's not already in your environment), and then place openaq.py in the same directory as your aq_dashboard.py file. Once you've included it in the folder, try running the following in a Python REPL to verify that you are able to make API requests using openaq.py. 

>>> import openaq 
>>> api = openaq.OpenAQ() 
>>> status, body = api.cities() 
>>> status
200
>>> body 
{'meta': {'name': 'openaq-api', 'license': 'CC BY 4.0', 'website':...
Requests to OpenAQ return a tuple of status (of the HTTP response - 200 is OK) and the body (the response payload, as a Python dict). The body at the top has a meta key for metadata, and a results key for the actual data. One of the more interesting endpoints is Measurements Links to an external site..

>>> # Assuming we have the api object from above
>>> status, body = api.measurements(city='Los Angeles', parameter='pm25') 
>>> status 
200
>>> body['meta'] 
{'name': 'openaq-api', 'license': 'CC BY 4.0', 'website': 'https://docs.openaq.org/', 'page': 1, 'limit': 100, 'found': 3069, 'pages': 31} 
>>> len(body['results']) 
100
>>> body['results'][:2] 
[{'location': '21 de mayo', 'parameter': 'pm25', 'date': {'utc': '2019-03-08T00:00:00.000Z', 'local': '2019-03-07T21:00:00-03:00'}, 'value': 8.13, 'unit': 'µg/m³', 'coordinates': {'latitude': -37.471182288689, 'longitude': -72.36146284977}, 'country': 'CL', 'city': 'Los Angeles'}, {'location': '21 de mayo', 'parameter': 'pm25', 'date': {'utc': '2019-03-07T23:00:00.000Z', 'local': '2019-03-07T20:00:00-03:00'}, 'value': 8.13, 'unit': 'µg/m³', 'coordinates': {'latitude': -37.471182288689, 'longitude': -72.36146284977}, 'country': 'CL', 'city': 'Los Angeles'}]
We are pulling 100 observations of measurements of fine particulate matter (PM 2.5) in the Los Angeles area (in Chile, not the US). Note that this is equivalent to making a request to this URL (you can even see the data in your browser!): https://api.openaq.org/v1/measurements?city=Los Angeles&parameter=pm25Links to an external site.

Run the above examples (you will get different data - it's a live API!) locally, and then incorporate the specific request api.measurements(city='Los Angeles', parameter='pm25') into your application as a function named get_results:

Import and set up the API object in your aq_dashboard.py file
Retrieve the data from the API when the main ('root') route is called
Create a list of (utc_datetime, value) tuples, e.g. the first two tuples for the data returned above would be: [('2019-03-08T00:00:00.000Z', 8.13), ('2019-03-07T23:00:00.000Z', 8.13)]
Return this list in the main route, so that the home page of the web application displays the raw list of tuples
Getting this list of tuples may be trickier than you think - the API returns dictionaries inside of dictionaries! You may want to use the REPL or notebook to experiment until you find working code, and then add it to your application.

Hint - Flask routes return strings, so if you call str() on your list of tuples to convert it, the entire resulting string can be returned by the route and displayed on the page..

Another hint - put the logic for pulling a processing results (into the list of tuples) in a separate function from the root() route - the route can just call it, cast the result to a string, and return that.

Part 3 - That Data Belongs In A Model!
As with reservations, taking data is great - but holding is the most important part. Links to an external site.Let's use flask-sqlalchemy to keep our data in a durable store –a database. Create a Record model - you can start by adding the following code to aq_dashboard.py:


from flask_sqlalchemy import SQLAlchemy

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///db.sqlite3'
DB = SQLAlchemy(app)

class Record(DB.Model):
    # id (integer, primary key)
    # YOUR CODE HERE
    # datetime (string)
    # YOUR CODE HERE
    # value (float, cannot be null)
    # YOUR CODE HERE

    def __repr__(self):
        return 'TODO - write a nice representation of a Record'

@app.route('/refresh')
def refresh():
    """Pull fresh data from Open AQ and replace existing data."""
    DB.drop_all()
    DB.create_all()
    # TODO Get data from OpenAQ, make Record objects with it, and add to db
    # YOUR CODE HERE
    DB.session.commit()
    return 'Data refreshed!'
Please finish the data model for the Record table creating table columns that have the attributes mentioned in the comments within the class (see code above).  
Then, complete the missing portions of the refresh route to loop over the list of tuples from your get_results function to add new records to the database. As the Record class indicates, store any datetime as a string –SQL does support native datetime objects.
While the provided Record class is adequate for storage, you should implement the __repr__ method to provide a nicer representation, so when the objects are printed or converted to a string you can clearly see their time and values.
To verify that these records are being correctly inserted in the the DB, you can execute FLASK_APP=aq_dashboard.py flask shell

Your output may look different than what is shown below if you have written your __repr__ function differently. But it should contain comparable information.

>>> from aq_dashboard import Record
>>> Record.query.all()[:5]
[< Time 2019-03-08T01:00:00.000Z --- Value 9.48 >, < Time 2019-03-08T00:00:00.000Z --- Value 3.0 >, < Time 2019-03-08T00:00:00.000Z --- Value 8.13 >, < Time 2019-03-07T23:00:00.000Z --- Value 3.0 >, < Time 2019-03-07T23:00:00.000Z --- Value 8.13 >]
Part 4 - Dashboard to the Finish
Now that your data is in a database, revisit your main route - instead of pulling all data directly from the API, query the database for any Record objects that have value greater than or equal to 18. The filter Links to an external site. method of SQLALchemy queries will be invaluable for this. Hint - your query should look like Record.query.filter(condition).all(), where condition is a comparison/statement that returns a boolean (true/false), and you can access the fields of Record to make that comparison.

Finally, return this filtered list of "potentially risky" PM 2.5 datetime/value tuples (again, you should make it a string for Flask). You now have a (very basic) dashboard, that stores, updates, and displays useful data! Note - you may get few or possibly even no values above the threshold. You can doublecheck by investigating the raw data, but that may be correct - it just means Los Angeles happens to have fairly clean air right now!

Part 5 - Turn it in!
Submit aq_dashboard.py and your db.sqlite3 file below.

Please also add a screenshot of your running application loaded locally in a web browser, to facilitate grading and feedback. Thanks for your hard work!

If you got this far, check out the the official OpenAQ data web application Links to an external site. - it looks impressive, but really it's just built on another API (OpenStreetMap Links to an external site.). Also, you can read up more on PM 2.5 Links to an external site. and how it relates to health.

Stretch goal (not part of the Sprint Challenge) - make it more usable! Put the results in a simple template (for loop with list items) so they look nicer, and add a link to trigger pulling the data.

Super stretch goals (also not part of the Sprint Challenge) - add a form so the user can specify which city to get a list of data for. Store data for different types of requests in the database, and connect the entities appropriately with relations. Instead of dropping all data for every pull, only add actually new data and keep the rest. Instead of just listing records that are above a threshold, do some actual data science on the data (averages, trends, models) and display those results. Then deploy it with Heroku!

None of these are realistic to do in a Sprint Challenge, but if you like this topic, feel free to run with it and make a personal project out of it. Just please don't share the base solution itself - thanks!

Good luck!
