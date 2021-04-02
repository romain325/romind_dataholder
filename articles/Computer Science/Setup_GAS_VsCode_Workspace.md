# Setup your Env for Google Apps Script dev  

## The Tools  

you'll need different tools to get the most efficient working environment.
There is a list of them:  
    - NPM (node package manager)  
    - clasp --> `sudo npm install -g @google/clasp`  
    - VsCode --> (or any editor you want)  
    - a web browser
    - a google account  

-----------------------------

## Create Project  

If you start from scratch then this part is made for you.  
First you'll create a new folder, and hit `clasp create <name>`  
If you're not logged into clasp, please refere to [[Use Clasp]]  
Now let's run `npm install --save-dev @types/google-apps-script`  
Create a file `.claspignore` and add node_modules and package-lock.json to it  
You can now run `code .` to open your folder in code and start coding.  

-----------------------------

## Getting an existing Project  

Create a new folder and run `clasp clone <id>` and install the dependencies (see [[Create Project]])  

-----------------------------

## Use Clasp  

First, you'll have to log into clasp, just run `clasp login` and follow the instructions.  
You can now run clasp push and pull as you desire (like every other repo)  
you don't need to add/commit like standart repo, just push !  
Your `.js` or `.ts` will automatically get transcribed to `.gs`  

-----------------------------  

## More

You can use any other tools on top of it, including git for example  
Just remember to add files/folders to .claspignore when they're not needed  
And add .clasp.json and appsscript.json to your .gitignore too !  
