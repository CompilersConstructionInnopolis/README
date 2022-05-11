Instructions for building the project 
For now we don’t have “ready to use scripts” that will build the entire project. We believe that we will implement it as soon as possible. In the current situation, to build our project you need to have maven installed on your machine.
 
Firstly, you need to clone a repository of our project.
Then, you need to navigate to the folder of the project using the terminal.
Finally, you need to run the command “mvn clean install”. This command will create a ‘target’ folder and an executable .jar file inside.
 
Instructions for using the prototype

To use a prototype, you need firstly create a build of the application (see previous step)
Then, you need to navigate to the created ‘target’ folder.
Finally, you need to run a command:
java -jar Compilers_Innopolis-1.0-SNAPSHOT.jar ../Tests/program.txt
	
	    Note: before running a command you need to enter a code of the program inside 		  Tests/program.txt file.
