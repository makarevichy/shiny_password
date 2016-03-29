#### **Shiny Password: A simple (and not particularly secure) method for managing user access permissions in Shiny apps**


RStudio's product Shiny allows users to easily build interactive web applications backed by an R session. Shiny applications can be served to end users via three mechanisms:

- [shinyapps.io](http://www.shinyapps.io/) (RStudio's hosting platform)
- [Shiny server open-source](https://www.rstudio.com/products/shiny/download-server/) (free and open source)
- [Shiny Server Pro](https://www.rstudio.com/products/shiny-server-pro/)


Both Shiny Server Pro and the highest two tiers of shinyapps.io allow administrators to set app-specific user access permissions. However, some users of shinyapps.io's lower tiers or Shiny server open-source may want to control app access as well.


The most secure method for accomplishing this with Shiny server open-source is to [host the app behind a reverse proxy server](https://support.rstudio.com/hc/en-us/articles/213733868-Running-Shiny-Server-with-a-Proxy) that will manage web traffic to and from the Shiny server. The reverse proxy server can then use standard user access control tools such as [Auth0](https://auth0.com/blog/2015/09/24/adding-authentication-to-shiny-open-source-edition/) or [PAM](https://en.wikipedia.org/wiki/Pluggable_authentication_module).


However, in some deployment environments using a reverse proxy may not be possible - this Shiny app template is designed for those scenarios. This app code conducts user authentication inside the Shiny app itself.


This approach does not resolve Shiny server open-source's use of an unencrypted web connection (it uses HTTP instead of HTTP**S**). A user password will be passed to the Shiny server in clear text, so anyone snooping on the web connection will be able to steal it.


***

#### **Usage**


This code is a template that can be used to add login functionality to any Shiny app. Users should place the code for their Shiny app in the appropriate locations in the template files.


The login interface operates by wrapping the Shiny app's entire UI specification in a renderUI() function. By default, an app user will only see the login screen. When he successfully authenticates, the conditional UI is triggered and the app content appears. This means that **the entire UI specification code is written in the server.R file** (as compared to it normally residing in the ui.R file). Your app's UI code should be written starting on line 16 of the server.R file.


The login template manages user credentials by saving them in a persistent data frame local to the Shiny app. The data frame contains 3 elements for each user: user name, password, and lock out status. Each time an app user attempts to log in, the app reads the data frame and compares the submitted credentials to the data frame's set of valid credentials. Note: the passwords are stored as md5 hashes so they are not legible to someone who managed to access the credentials data frame - this likely means the app owner will have to assign a new password to a user who forgets his since it can't simply be recovered from the credentials data. 


The app owner may specify how many times a given user can attempt to log in before he is locked out by adjusting the "num_fails_to_lockout" parameter in the global.R file. Once a user fails to log in that number of times, his/her lockout status is set to TRUE in the persistent credentials data frame. Thus, if he reconnects to the app in a new session he will still be locked out. If a user fails to login too many times without specifying a valid user name, the session is locked out (but if the user closed the connection and reconnected he could continue attempting to login).


The admin.R file contains functions that the app owner can use to manage the user credentials data. When the app is first deployed, the credentials_init() function should be called. It creates the "credentials" directory in the app's local directory and saves an empty credentials data frame with the correct structure there.


The app owner may add one user to the credentials data by using the add_one_user() function or add multiple users by using the add_multiple_users() function. The former adds a single entry to the credentials data; the user name and password are submitted as character strings. The latter adds multiple entries to the credentials data - the user names and passwords are submitted as two character vectors, with each user name paired with the entry with the same index in the password vector.


The app owner may delete one user from the credentials data by using the delete_user() function with the user name specified - this function only deletes one user per call.

***

#### **WARNING AND DISCLAIMER:**
**I am not a security professional. This app template provides an *extremely* mild form of access control that can be easily circumvented by a knowledgable adversary.**
