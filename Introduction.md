# Introduction #

Creating services in the Go language is straight forward, GoRest takes this a step further by adding a layer that makes tedious tasks much more automated and avoids regular pitfalls. This gives you the opportunity to focus more on the task at hand.

# Why GoRest #

The RESTful architectural style promises a lot for scalable web applications.
Such applications are easily built on top of just basic HTTP API’s or maybe simple RESTful frameworks.

This kind of approach to the development of RESTful webservices is valid but has a lot of limitations when applications start getting bigger and complicated.
One could compare this first approach to the writing of simple dynamic web pages using Servlet/CGI/ or even JSP/ASP/etc; versus the approach of using higher level frameworks like Rails/Struts/etc.

The higher level frameworks provide more manageability, predictability and thus sustainable reliability of such projects.

GoRest takes the latter approach of letting the framework handle as much as possible of the predictable no-brainers in the stack. Things that would probably end up being repeated over and over by every app are all integrated within GoRest. This also helps in avoiding common pitfalls at both development and runtime; resource path routings, authorization, errors, etc are all handled predictably.



# Design Goals #

The main design goal is to provide an environment that gives the developer a _natural-feeling_ when coding, not obscured by HTTP specifics.

GoRest is mostly designed for the programmable web rather than normal/readable web, but works well for both… i.e: allows good exchange of state(JSON/XML/etc) rather than presentation(HTML).
It is therefore programmer driven rather than designer driven, hence it encourages features like type safety and a simple path routing system that is reliable, specific and predictable  (does not rely on regex or some fuzzy logic). We feel it is more secure this way.