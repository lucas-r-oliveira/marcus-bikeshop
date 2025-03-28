# 1.  Marcus Bikeshop
Marcus Bikeshop is Marcus' online e-commerce store for his bike shop business. 
It offers a catalog of bicycles (and possibly in the future other sports equipment), which the client can fully customize.
It allows Marcus to customize which parts and which options for the different parts are provided per Bike (Product).

This project is for a technical exercise.

# 2. Installation & Setup
⚠️ Before cloning the repository ⚠️ <br>
This repository is composed of other git repos. That means this repo is composed of git submodules. So, to clone the repo you need to:
```
# Clone the repository
git clone --recurse-submodules https://github.com/lucas-r-oliveira/marcus-bikeshop.git
```

And since everything is dockerized, to run the project you need to make sure docker is installed on your machine:
```
docker compose up
```

⚠️ Do note: At the time of writing this, the application still has bugs and missing features. Please keep that in mind before running the project.

# 3. Architecture
For this project I went for a Domain-Driven Design (DDD) approach. 
### What is DDD?
"Domain-Driven Design is an approach to software development that centers the development on programming a domain model that has a rich understanding of the processes and rules of a domain." - [Martin Fowler](https://martinfowler.com/bliki/DomainDrivenDesign.html)

Essentially, the domain is the problem we're trying to solve and the domain model is the mental map that business owners have of their businesses. 

### Why DDD?
DDD allows us to capture the requirements of the business into a dependency-free layer in our application. This is important because: 
1) It gives us a ubiquitous language that we can use to communicate with the stakeholders and validate our assumptions.
2) The business requirements should dictate the technical needs of our application and not the other way around. 
3. By having the business logic layer free of dependencies and technical constraints, we can refactor aggressively if necessary.
4) It allows us to introduce patterns and abstractions that would otherwise be difficult. 

### How was DDD applied to this app?
#### The Domain Model
First things first, a brief introduction to some DDD concepts:

<img alt="domain objects" src="https://github.com/user-attachments/assets/c5d2400e-d65e-4b32-92db-54f1d682cae3" width="500">

- Entity - Used to describe a domain object that has a long-lived identity.
- Value Object - A _value object_ is any domain object that is uniquely identified by the data it holds; These are usually immutable.
- Aggregate - An _aggregate_ is just a domain object that contains other domain objects and lets us treat the whole collection as a single unit. Each aggregate is usually associated with a Repository (described two subsections below).
- Domain Service - Something that is neither an Entity nor a Value Object. Not to be confused with Service Layer services.


Below we see the domain model of this business boiled down to its absolute essentials:

<img alt="domain model" src="https://github.com/user-attachments/assets/2c3681ef-1f9e-4666-9ab8-36278842d274" width="500"><br>


If we consider the domain of this application to be e-commerce, we can then divide it into multiple subdomains. A possible example:

<img alt="e-commerce domain" src="https://github.com/user-attachments/assets/caab4168-dae6-4023-abb5-f952fa115337">


[Reference](https://simonatta.medium.com/e-commerce-by-ddd-bf4459272188)

For this app we've considered 3 subdomains only:
- Product (Catalog)
- Orders
- Configuration Rules

#### The overall architecture
This was the Architecture achieved. 

![overall architecture](https://github.com/user-attachments/assets/99b174ac-8d22-4866-9636-3c61b934897a)

We can see the dependency flow is top-to-bottom.
But let's go over this bottom-to-top.

**Domain**
We've been over this in the previous section. This is where we define our domain objects and, hence, try to capture the business logic of our app.

**Repositories**
The Repository Pattern allows us to decouple our domain from the data layer by introducing an abstraction in the middle. The Repository uses a collection-like interface for our domain objects.  This pattern shows to be extremely convenient when it comes to testing, for example. With this approach we can easily switch out our entire data layer, so we can be very selective on the things we actually want to test.
In this diagram, in particular, we can see an Abstract Repo and a SQLAlchemy* Repo that implements the former. This is our production flow. If we wanted to focus on testing, instead, we could easily switch out our SQLAlchemy Repo by an In-Memory Repo.

**Service Layer**
The Service Layer is responsible for orchestrating our workflows and defining the use cases of our system. It acts as an integration layer. Services from this layer are free to interconnect with each other and even depend on each other.

**Flask**
Flask (API) acts as the entrypoint for the entire application. It is responsible for instantiating our database (if we have any) and the appropriate repositories to interact with it. It also instantiates our Service Layer services, on which we inject the Repositories. 
That's also where we define our views. Each subdomain has its own blueprint, i.e., it's split into its own collection of routes and views.



\*SQLAlchemy is an ORM library for Python. More info in section Tech Stack > Backend
#### Next steps?
So far, we've achieved an interesting architecture that helps us plenty already.
This architecture makes it easier for us to move towards an Event-Driven architecture, if we so desire.

For immediate improvements, we can easily spot that our API layer has **way too many** responsibilities. By adhering to the Unit of Work pattern, we can mitigate this effect. It allows to fully decouple our service layer from the data layer.

### So... Why NOT DDD?
So far, this all sounds great, but every decision has its tradeoffs and, naturally, DDD isn't an exception.
So, what are the tradeoffs considered here?

#### Overhead for simple apps
DDD introduces a lot of abstractions and patterns and introducing them in your code does not come for free. It does not contribute to **not** adding complexity to our codebase. It does, however, allow us to choose **where** we want to add complexity.

For a simple app, you could argument that DDD introduces way too much complexity and overhead for the benefit it gives. And, generally, I would agree with that statement.
You could also make the argument that this app is simple enough that we might not need DDD. I do agree with it to an extent. However, the requirements provided are not that of a fully-fledged app. So even for the simple version of a fully functional app that represents this business, we're missing out on a lot of (sub)domains. e.g.: Payments, Authentication, Shipping, Inventory, etc. Hence, if this were a real app, I believe that DDD is a strong architecture that helps us ensure our application is scalable and maintainable, by having our layers loosely coupled. If we only consider this exercise alone, then yes, DDD can bring quite the overhead.

#### Learning curve
DDD brings a new way of working. It introduces a lot of concepts (Entities, Value Objects, Bounded Contexts, ...) so, naturally, there's a learning curve associated to it. 
It is a skill and, as with any skill, it takes time to master.

Personally, even though I use it in my day-to-day work, it's different when you apply it to a domain you're already familiarised with, than when you're applying it from scratch.

It takes a big time investment to capture and model the business requirements just right. If you also apply TDD, a lot of trial and error is involved there as well. 
Here, it wasn't any different and that was a pain point for me.

# 4. Tech Stack

Overview
```
- Backend: Flask (Python)
- Frontend: React 18 (Typescript)
- Database: For now SQLite, but maybe PostgreSQL if I have the time
- State Management: React Contexts
- API: REST (OpenAPI)
- Deployment: Docker
```

### Backend
As much as I would love to have an excuse to try out Ruby on Rails, I ended up choosing Python + Flask. Python is the language that I'm the most comfortable with and, therefore, allows me to move faster. It also enables me to think in a more architectural way, because I don't need to be concerned about the constraints of a language that I don't know. 
When it comes to web app frameworks, I would say the 3 main ones for Python are: Django, Flask and FastAPI. Either of them would be a good choice for this use case, but let's go over each one of them in detail:
- **FastAPI** is known for its ease of use, documentation features and performance. If this application was really performance-critical, I would consider it, but since it was not and I don't quite know how well it scales for bigger applications, it was an easy one to kick out.
- **Django** has lots of great features. It has a lot of batteries included when you start an application and it has an extensive set of documentation. It also scales quite nicely. However, it relies heavily on the Active Record pattern. You don't have to use it, though you miss out on a lot of Django's strengths (ORM, Managers and Querysets) if you don't. It also makes it more complex to move if you don't use Django the way it was designed to be used. 
- Finally, we're left with **Flask**. Contrary to Django, Flask isn't tightly coupled with an ORM. In fact, it doesn't have one by default. Flask also doesn't have that many batteries included from the start, which can make it more painful when first starting a project, but it makes up for it by giving you more architectural freedom when designing an application.
 
I ended up going with Flask, because of the reasons stated above and because I'm quite well familiarized with the integrations around it.

As mentioned above, Flask also doesn't include an ORM by default. We can address that by using SQLAlchemy, for example, which is an ORM library for Python.

### Frontend
In the frontend world there's plenty to choose from, so I mainly went for familiarity. Even though I use Angular+Typescript in my day-to-day job, I much prefer the React+Typescript combo. I specifically used React 18, because I haven't yet tried the new changes of React 19. 

As I was dealing with some level of complexity in the backend, I decided to use the basics in the frontend.
That means that for state management I used React Contexts and Local Storage.
For routing I went for React Router.
Since the exercise specifically mentioned there was no need to focus on design, I also did not use any specific component library. Just plain HTML and CSS.

### Database
For the needs of this application, a relational database is sufficient. We have structured data, and no need for hyper-scalability. My go-to is usually PostgreSQL, because it's open-source and one of the most robust and complete RDBMS out there. 
However, SQLite is usually my go-to to get the foot on the door. Due to time constraints, the Database is currently stored in a sqlite file.


# 5. Testing approach
There is definitely room for improvement here...

Even though, it is not reflected in this repository, I started out with a Test-Driven Development (TDD) approach. It allowed me to freely experiment and give structure to my domain model. I spent quite some time writing unit tests and modelling my domain to match my test cases.

After some time and some rounds of iteration, I wasn't yet quite satisfied with how my domain model was turning out, but I had to move on. So I turned to other parts of my application, due to time constraints.

After making significant advancements in the frontend and backend, I came back to testing my application. Naturally, I didn't want to leave it untested.

Again, due to time constraints, I ended up focusing my tests on the Service Layer. As this layer acts as an integration layer between all other layers, I figured it was what allowed me to test my workflows and business logic as much as possible with as little possible effort.
The idea was also to at least include e2e tests, but it's still on the table.

In an ideal scenario, we would have a health balance of unit, integration and e2e tests. For this app this wasn't possible.

Like DDD, TDD is also a skill which takes time to master and definitely a skill I need to practice more.

⚠️ Instructions on how to run the tests will follow soon!

# 6. References
- https://martinfowler.com/
- https://www.cosmicpython.com/book/preface.html
