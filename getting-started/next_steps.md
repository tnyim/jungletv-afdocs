# Leaping _frogward_

Now that you have completed two guides to creating example applications, and perhaps even done a bit of off-rails exploration by yourself, you may be wondering about what it takes to create an original application based on your own ideas.
Firstly, we recommend giving the full [JungleTV AF manual](../manual/) a read, even if just a quick one, to become aware of more JAF concepts, capabilities and guidelines.
Then, you should confirm whether your ideas are feasible, taking into account the current state of the framework, and the vision the JungleTV team has for the service.
The best way to do this, is to [get in touch](../README.md#get-in-touch) with the team and discuss your ideas.

Once you are ready to start development, you should think about how the application will be organized, both in terms of code and in terms of user interaction.
You should probably also begin thinking about implementation details, such as how to structure the communication between the server and any application pages.
Do not be ashamed to ask for help, or to work based on example projects shared by others - just make sure to respect their licenses and copyrights.

For more complex projects, we recommend planning ahead with regards to how to test the application.
You will likely want to structure your code in such a way as to allow automated testing, and you may want to add some faculties specifically to make manual testing and debugging easier.
For example: if your application is meant to present some special behavior once every few hours, you will probably want to make sure you can trigger that special behavior more frequently while testing.
You may even want to assemble a quality assurance team of sorts, essentially people who will participate in testing sessions you may organize, and give feedback about the application.

When you reach a point where you believe your application is sufficiently complete and polished, it is probably the time to submit it to the formal [review process](../manual/review_deployment.md).
This is the first step towards having your application live on the production version of JungleTV, to be seen by hundreds of users.
By this point, you should make sure you understand exactly what rights you and the JungleTV team have over your application.
Worry not, the short summary is that while the JungleTV team reserves some rights, including to modify the application, ultimately the applications belong to their creators and you are able to back out of the agreement at any point.

---

This concludes the guide to getting started with the JungleTV Application Framework.
We hope it was sufficiently complete to get you started without overwhelming you with all the different concepts and capabilities.
Keep in mind that the JungleTV AF is not complete, so this ride is only getting wilder!

The next major section in this documentation is the manual, and as explained above, we recommend reading it to get a better idea of how the JungleTV AF works and what you can do with it.
The manual also provides answers to questions such as ["Why provide this complicated system instead of a regular public API over HTTP?"](../manual/architecture.md#the-missing-regular-api).