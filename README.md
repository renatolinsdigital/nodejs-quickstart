
# Node.js quickstart - Essential notes by Renato Lins

Node.js was initially created by Ryan Dahl in 2009 with the specific purpose of enabling server-side scripting using JavaScript. The intention was to develop a runtime environment that could execute JavaScript code outside of a web browser, allowing developers to use JavaScript for server-side programming. Since its creation, Node.js has evolved to become a powerful and widely-used platform for building scalable and high-performance web applications, enabling JavaScript to be used not only for server-side development but also for a variety of other applications, including desktop applications and IoT solutions.

---

### 1. Considerations about Node.js

Learning Node.js can significantly boost your career prospects in the tech industry. It's in high demand across various sectors due to its versatility, scalability, and performance capabilities. Mastering Node.js opens doors to opportunities in web development, microservices architecture, real-time applications, and remote work. With a growing ecosystem, strong community support, and adoption by leading companies, Node.js skills can lead to competitive salaries and future-proof your career in the ever-evolving tech landscape.

Key Features of Node.js:

__Asynchronous and Event-Driven__: Node.js uses an event-driven, non-blocking I/O model, which makes it lightweight and efficient. This is particularly well-suited for applications that require a lot of I/O operations, such as web servers and real-time applications.

__Single Programming Language__: With Node.js, developers can use JavaScript for both client-side and server-side development. This streamlines the development process and allows for more cohesive and maintainable codebases.

__High Performance__: Node.js is built on the V8 JavaScript engine, developed by Google for the Chrome browser. V8 compiles JavaScript directly to machine code, which results in fast execution times.

__Scalability__: Node.js is designed to build scalable network applications. It can handle many connections simultaneously with high throughput, making it ideal for applications that require handling many simultaneous requests, such as APIs and microservices.

### 2. Node.js Architecture overview

Understanding Node.js architecture is crucial for developers aiming to craft high-quality applications. It aids in performance optimization, effective debugging, dependency management, security, and best practices adherence. Some essential parts of Node.js architecture are:

__V8 Engine__: The V8 engine is developed by Google and is written in C++. It is responsible for executing JavaScript code in the Node.js environment. V8 compiles JavaScript code into machine code directly, which improves performance. It also manages memory allocation and garbage collection.

__Libuv__: Libuv is a multi-platform library with origins in the Node.js project. It abstracts operating system-specific functionality related to asynchronous I/O operations, file system operations, and networking. Node.js integrates Libuv to handle asynchronous operations efficiently, providing a unified interface across different operating systems. Libuv is crucial for Node.js's event loop mechanism, as it provides the underlying infrastructure for managing events, timers, and callbacks.

__Event Loop__: The event loop is at the core of Node.js's asynchronous, non-blocking I/O model. It continuously checks for events and executes corresponding callbacks when events occur. The event loop operates in phases such as timers, I/O polling, setImmediate, and close callbacks. Each phase handles different types of events and callbacks.

__Event Queue__: Events and their associated callbacks are queued in different event queues based on their type. For example, I/O-related callbacks go to the I/O event queue.

__Call Stack__: The call stack is part of the V8 engine and is responsible for managing the execution context of JavaScript code. Each function call creates a stack frame that contains information about the function's execution, including local variables and arguments.

__LIFO Structure__: The call stack operates on a Last-In-First-Out (LIFO) basis, where the most recently called function is processed first.

__Asynchronous Operations__: When Node.js encounters asynchronous operations, such as I/O requests or timers, they are offloaded to the event loop and do not block the call stack. Once the asynchronous operation completes, its callback is added to the event queue and eventually processed by the event loop.

__HTTP and TCP/UDP__: Node.js provides built-in modules like http, https, net (for TCP), and dgram (for UDP) for handling network protocols. Developers can create HTTP(S) servers using the http or https modules, handle incoming requests, and send responses asynchronously. These modules leverage Node.js's event-driven architecture, allowing efficient handling of network events like incoming requests, data transmission, and connection management.

__NPM (Node Package Manager)__: NPM is bundled with Node.js and serves as a package manager for installing, managing, and sharing JavaScript packages. Developers define dependencies and metadata in the package.json file, which NPM uses to manage package installations and versioning.

__Ecosystem__: NPM provides access to a vast ecosystem of open-source libraries and tools, allowing developers to leverage existing solutions and accelerate development.

__Native Add-ons__: Node.js allows developers to create native add-ons using C/C++ to extend its capabilities or integrate with existing native libraries. Native add-ons are beneficial for performance-critical tasks that require low-level system access or integration with hardware.

__Bindings__: Node.js provides bindings that enable communication between JavaScript code and native add-ons, facilitating seamless integration and interoperability.


### 3. Running Node.js locally

When we install Node.js on our machine, it essentially enables JavaScript to communicate with our system directly. This means we can do things like automate tasks, create servers, manage files, and interact with databases using JavaScript. It expands the reach of JavaScript beyond just web browsers, allowing us to unlock the full potential of Javascript. Let's now delve into how we can harness JavaScript's power on our machines with the assistance of Node.js.

1. __Installing Node.js__

To get started with Node.js, follow these steps:

a. Navigate to the official Node.js website at __nodejs.org__.

b. Click on the download button to access the suitable version of Node.js for your operating system (Windows, macOS, or Linux). You'll have the choice between the LTS (Long-Term Support) version, offering the most recent stable release and recommended for most users, and the version with the latest features, suitable for experimental, advanced, or specific needs.

c. Once the download is complete, run the installer and follow the on-screen instructions to install Node.js on your computer.

d. During the installation process, ensure to include the option to add Node.js to your system's PATH. This will make it easier to run Node.js commands from the terminal.

e. After installation, open the terminal (Command Prompt on Windows, Terminal on macOS and Linux) and type ```node --version``` to confirm that Node.js was installed correctly and to check the version.

Now, you can execute Node.js scripts directly from the command line using the node command. This will launch the Node.js interpreter, denoted by the > prompt. From here, you can execute JavaScript commands. For instance, typing console.log('Hello, Node.js!') will output 'Hello, Node.js!' in the terminal. To exit the Node.js interpreter, type ```.exit``` or ```Ctrl + C``` twice.

2. __Installing Visual Studio Code and Essential Plugins for JavaScript Development__

Visual Studio Code (VS Code) is a powerful code editor developed by Microsoft, known for its versatility and lightweight design. To install VS Code, visit __code.visualstudio.com__, download and run the installer for your operating system, then follow the on-screen instructions to complete the installation process.

Essential VS Code Plugins for JavaScript Development:

__Prettier (by Prettier):__ Prettier is a versatile code formatter that supports various languages like JavaScript, HTML, CSS, TypeScript, JSON, Markdown, and more. It can be used directly within code editors with extensions, through command-line tools, as pre-commit hooks in version control systems, or integrated into build tools and CI pipelines. Its purpose is to automate code formatting, ensuring consistency and readability across projects.

__Path Intellisense (by Christian Kohler):__ Path Intellisense offers autocomplete functionality for file paths within import statements and HTML src attributes, aiding in efficient navigation within your project. Its suggestions are automatically populated after installing the plugin, helping you save time by accurately predicting file paths as you type.

__Live Server (by Ritwick Dey):__ Live Server launches a local development server with live reload functionality for HTML, CSS, and JavaScript files. This tool is typically used to create a local server for running front-end projects, especially when JavaScript code is associated with HTML files. Its usage is extremely simple: just right-click on the index.html file (for example) and select 'Open with Live Server'.

__JavaScript (ES6) code snippets (by charalampos karypidis):__ Provides a collection of ES6 code snippets for faster coding. For example, once it is installed, we can type ```mde``` and hit TAB so it will generate the code snippet ```module.exports = { };```

__Code Spell Checker(by Street Side Software)__:  This extension helps maintain clean and error-free code by highlighting spelling mistakes in comments and strings. It assists in avoiding typos and ensuring code professionalism.

General Configuration: Visual Studio Code allows you to customize settings, keybindings, and themes to suit your preferences. Explore the various features provided by VS Code and its plugins to enhance your JavaScript development experience.

3. __Automatic code formatting with Prettier__

To enable automatic code formatting with Prettier upon saving JavaScript files, create a folder named __.vscode__ in the root directory of your project. Inside this folder, include a file named __settings.json__ with the following configuration:

```json
{
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  }
}
```

Once configured, you'll notice that your JavaScript code is automatically formatted according to Prettier's rules whenever you save a file. This includes consistent indentation, proper spacing, and other style guidelines, leading to cleaner and more maintainable code. These automated adjustments streamline your development process, eliminating the need for manual formatting and ensuring code consistency across your projects. To customize Prettier's rules, you can create a __.prettierrc__ file in your project's root directory. This file allows you to specify your preferred formatting rules for Prettier. Here's an example of a .prettierrc file:

```json
{
  "singleQuote": true,
  "semi": true,
  "tabWidth": 2,
  "printWidth": 100
}
```

In this example: __"singleQuote"__ Configures Prettier to use single quotes instead of double quotes for strings. __"semi"__ specifies whether to add semicolons at the end of statements. __"tabWidth"__ sets the width of a tab character. __"printWidth"__ specifies the maximum number of characters per line before code is wrapped. 

To explore more options and documentation for Prettier, you can visit __prettier.io__

4. __Running Javascript code with Node.js capabilities__

With Visual Studio Code open, navigate to the desired folder (where your project will be), right-click, and select 'New File'. Name it with a .js extension (e.g., __scripts.js__). Inside this file, you can write your JavaScript code, like the following:

```javascript
const num1 = 10;
const num2 = 20;
const result = num1 + num2;
console.log(`The sum of ${num1} and ${num2} is: ${result}`);
```

Once you're in the directory containing your JavaScript file, you can run it using Node.js. Open your terminal, navigate to the directory, and type ```node scripts.js```. This command will execute your JavaScript file and display the output in the terminal. 

Node.js comes with a built-in module called File System (fs), which enables various operations such as dealing with the file system. For instance, if we want to create a .txt file and write to it, we can use the __fs__ module. It provides a way to write to a file while passing a function to handle possible errors. The __fs__ module in Node.js stands for "file system" and allows performing operations like reading from and writing to files, creating directories, deleting files, and more. As one of the core modules in Node.js, it comes pre-installed and doesn't need separate installation via a package manager. In Node.js, we use the ```require``` function to include modules in JavaScript files. When we use ```require('module_name')```, Node.js searches for the specified module in the file system, loads and evaluates it, and returns its exported object. The ```require``` function is part of the CommonJS module system, specifically designed for server-side JavaScript environments like Node.js, where modules are loaded synchronously. With this knowledge, we are fully capable of understanding, writing, and expanding upon code such as the following:

```javascript
// loads the "file system" module
const fs = require('fs');

// Define the file name
const fileName = 'example.txt';

// Text to be written to the file
const textToWrite = 'Hello, world! This is some text written to a file using Node.js.';

// Write to the file
fs.writeFile(fileName, textToWrite, (err) => {
    if (err) {
        console.error('Error writing to file:', err);
    } else {
        console.log('File successfully created and text written.');
    }
});
```

This code loads the built-in "file system" module (fs) in Node.js, defines a file name as example.txt, prepares a string containing some text, writes the content of the string to the file specified by fileName, and logs either an error or success message based on the writing process. To learn more about what can be done with Node.js, including its built-in modules like "file system" (fs), you can refer to the official Node.js documentation available on their website: __https://nodejs.org/docs/latest/api__. Additionally, there are various tutorials, courses, and online resources dedicated to learning Node.js that can provide comprehensive information on its capabilities and usage.

Ps. While loading modules using the ```require``` function is the traditional approach in Node.js, we'll also explore how to achieve this using __ES Modules__ later on.

5. __Understanding Node Package Manager (npm)__

Node Package Manager (npm) is a powerful tool used to manage JavaScript packages and dependencies for Node.js projects. npm is the default package manager for Node.js and JavaScript projects. It allows developers to easily install, publish, and manage packages and dependencies for their Node.js projects. These packages contain reusable JavaScript code, libraries, frameworks, and tools that enhance development productivity and extend the functionality of Node.js applications.

Benefits of Using npm:

* Efficient Dependency Management: npm simplifies the process of managing dependencies by automatically installing and updating packages required for your project.

* Large Package Repository: npm provides access to a vast repository of packages, offering a wide range of functionality for various development needs.

* Version Control: npm allows developers to specify and manage package versions, ensuring consistency and compatibility across projects.

* Community Support: Many npm packages are actively maintained by the open-source community, providing continuous improvements, bug fixes, and support.

* Integration with Build Tools: npm integrates seamlessly with popular build tools and task runners like webpack, Gulp, and Grunt, enhancing the development workflow.

Installation of npm: npm typically comes bundled with Node.js installation. When you install Node.js on your computer, npm is automatically installed along with it. Therefore, to install npm, you need to install Node.js first.

6. __Installing and managing dependencies__

To install an npm package, open your terminal or command prompt, navigate to your project directory, and run the command ```npm install <package-name>```. For example, to install a popular 
JavaScript utility library called __lodash__, consider running ```npm install lodash``` or simply ```npm i lodash```. So we have some things to consider:

__Associated files and folders:__ npm automatically manages dependencies for your project by creating a __package.json__ file that lists all installed packages and their versions. You can also specify dependencies manually in this file and use npm install to install them. The __package-lock.json__ file is created by npm to lock down the exact version of each package and its dependencies installed in a project. It ensures consistency across different installations by guaranteeing that everyone working on the project uses the same versions of dependencies. This file is automatically generated and updated whenever npm installs or updates packages in the project. The __node_modules__ folder is where npm installs project dependencies. It contains all the packages and their dependencies specified in the package.json file. Each installed package is stored in its own folder within node_modules, ensuring that the project has access to the required modules. This folder is automatically created and managed by npm when packages are installed using commands like npm install. It's essential for storing and organizing all the external libraries and modules used in a Node.js project.

Note: When uploading your project to GitHub, it's crucial to ignore the 'node_modules' folder. Other developers can re-download this folder on their machines. Additionally, ensure to upload both __package.json__ and __package-lock.json__ files, as they specify the library versions required for the project.

__Using installed packages:__ After installing a package, you can import and use it alongside your JavaScript code. For example, if you installed lodash, you can import it using ```require('lodash')``` for example. With your __scripts.js__ file you can test the installation and usage of lodash with the following code:

```javascript
const _ = require('lodash');

// Sample array
const numbers = [1, 2, 3, 4, 5];

// Double each element using lodash map
const doubledNumbers = _.map(numbers, num => num * 2);

// Output the result
console.log(doubledNumbers); // Output: [2, 4, 6, 8, 10]
```

To manage our dependencies we want to consider some key aspects:

__Installing packages for Development:__ When you install a package with the ```--save-dev``` flag or the ```-D``` shorthand, npm adds the package to your __devDependencies__ inside the __package.json__ file. These dependencies are only required for development or testing purposes, not for running the application in a production environment. For instance, if you install a testing framework like Mocha with ```npm i mocha -D```, it will be listed under devDependencies because it's only needed during development, not when deploying the application to a production server. 

Note: In npm v5+, installing a package automatically adds it to package.json. Using ```npm install <package-name>``` saves the package by default. However, if no package.json exists, using ```--save``` is recommended to make sure dependencies are explicitly saved. For example, using ```npm i lodash --save``` can clarify intentions and maintain consistency.

__Uninstalling dependencies:__ We can uninstall dependencies using ```npm uninstall <package-name>``` or ```npm uninstall <package-name1> <package-name2> <package-name3>``` for multiple dependencies. When using ```npm uninstall <package-name>```, npm removes the package from both the node_modules directory and the package.json file by default. 

__Listing installed dependencies:__ You can check all installed packages and their versions by running the command ```npm list```.

With npm, we can incorporate libraries to run alongside our project's automated processes. For example, we can download a specific library and use it as our code linter by running ```npm run lint```. While configuring this type of usage is beyond the scope of this tutorial, it's not difficult to set up and plenty of examples can be found online. Additionally, you can use ```npm help``` to discover more commands enabled by npm. By leveraging npm, you can streamline your Node.js development process, efficiently manage project dependencies, and access a vast ecosystem of JavaScript packages and tools.

7. __Using Yarn or other package managers__

Yarn, developed by Facebook is known for its speed, efficiency, and user-friendly nature. It ensures consistent installations and expedites package management through caching techniques. Yarn also facilitates concurrent task execution and provides clear error messages. npm, a cornerstone since 2010, comes bundled with Node.js and boasts a vast library of packages, valued for its straightforward installation process.

In general, Yarn tends to consume more disk space compared to npm due to its caching mechanism, which locally stores packages to expedite installations. This caching can result in larger overall disk space usage over time, particularly with frequent package installations or updates. Conversely, npm typically uses less disk space by default, though it may also consume additional space if caching features are enabled or if there is a high volume of dependencies.

__Package Management:__ Yarn employs a refined approach to ensure deterministic package version resolutions.

__Speed:__ Yarn often outpaces npm, thanks to its efficient caching mechanism.

__Security:__ While both prioritize security, Yarn offers additional features that bolster security.

__Workspaces:__ Yarn supports workspaces, enabling management of multiple packages within a single repository, particularly beneficial for larger projects.

While Yarn excels in faster installations, better dependency resolution, and enhanced security features, it tends to be favored in larger projects with more complex dependencies. On the other hand, npm is valued for its simplicity and broad adoption, making it a common choice for smaller projects or those with storage constraints.

Installing and using Yarn:

To install Yarn, you can use a package manager like npm or Homebrew (for example). If you're using npm, you can install Yarn globally by running ```npm install -g yarn```. Once installed, you can use Yarn commands in your terminal.

Yarn commands closely mirror npm's functionality. For example, use ```yarn add <package_name>``` to install a package, ```yarn remove <package_name>``` to uninstall, ```yarn list``` to view dependencies, ```yarn add <package_name> --dev``` for development dependencies, ```yarn upgrade <package_name>``` to update a package, ```yarn run <script_name>``` to execute custom scripts from your package.json file, and so on.

If Yarn appears to better suit your project needs, adopting it doesn't necessitate a permanent commitment or an exclusive reliance on it. Certain situations may arise where projects are tightly integrated with package managers tailored to specific operating systems. For instance, on Windows, Chocolatey, or on macOS, Homebrew may be preferred, particularly when managing JavaScript packages system-wide.

Furthermore, it's crucial to recognize that technology continually evolves, leading to the emergence of new package managers with enhanced user-friendly features. An example of this is pnpm, an alternative to npm, boasting benefits such as efficient dependency storage and performance enhancements. Thus, remaining open to such advancements ensures staying abreast of the latest tools and practices in software development.

8. __Adding 'hot reload' with the help of nodemon__

Consider a scenario where you aim to use Yarn and automate the execution of your JavaScript code every time you save a file. Initially, you need to install Yarn globally using the command ```npm install -g yarn```. Next, install a package that monitors file changes and triggers execution using Node.js. You can achieve this by adding nodemon as a development dependency with ```yarn add nodemon -D```. Once the necessary dependencies are installed, configure a 'start' script in your package.json file to watch and execute your scripts upon saving. This can be done by adding a 'scripts' entry to package.json:

```json
"scripts": {
  "start": "nodemon scripts.js"
}
```

Here's where the 'magic' happens: running ```yarn start```. From now on, any changes made to scripts.js and saved will automatically trigger our code to rerun. Pretty neat, right? To stop watching, simply navigate to the terminal and press Ctrl + C.

9. __Installing and Managing Multiple Versions of Node.js with NVM__

In certain scenarios, managing multiple projects with varying Node.js versions can become cumbersome. Reinstalling different Node.js versions or manually updating environment variables for each project switch isn't practical. To address this challenge, tools like Node Version Manager (nvm) provide a solution by simplifying the installation and usage of any Node.js version released to date. Hence, we can explore:

__Installing NVM on Windows:__ Download and run the NVM installer for Windows from the following link: __https://github.com/coreybutler/nvm-windows/releases__. Look for an installer like __nvm-setup.exe__ and proceed with the installation as usual.

__Installing NVM on Linux:__ Visit the NVM project on GitHub: __https://github.com/nvm-sh/nvm__. Follow the instructions in the README file to get the commands for the installation of the latest NVM version.

After the installation, you can verify the NVM Version by running ```nvm --version```. Now we have some possibilities around Node.js installation management:

* Installing the latest version: ```nvm install node```
* Setting the newly installed version as the default one: ```nvm use node```
* Installing a specific version, for example, version 14: ```nvm install 14```
* Using a specific version, for example, version 14: ```nvm use 14```
* Uninstalling a specific version, for example, version 14.21.3: ```nvm uninstall 14.21.3```
* Setting the default version (so that whenever you open a new terminal window the specified version will be activated): ```nvm alias default <version>```
* Checking the Node.js and NPM installation versions: ```node -v``` and ```npm -v```
* Listing the installed versions of Node.js: ```nvm list``` or ```nvm ls``` (The currently active version will be marked with an asterisk)

Note: If the ```nvm use``` command has never been run, no version will be used, even if Node.js has already been installed on your machine.

10. __Documenting shareable projects that use Node.js__

If you intend to upload or share your project on a Git platform, you should instruct others to:

* Download or clone the project repository
* Ensure that Node.js is installed on their machines, which also includes npm
* Install Yarn or any other package manager if the project does not use npm
* Install the dependencies listed in the package.json file. For example, by running ```npm install``` for npm or ```yarn``` for Yarn
* Launch the project. If scripts are not defined in the package.json file, the project is typically executed with ```node <script_name>.js```. However, if a 'start' script is configured and Yarn is being used, the project can be started with the ```yarn start``` command

### 5. Module Systems in Node.js: CommonJS and ES Modules

Node.js supports two primary module systems: CommonJS and ES Modules. Each system has its own syntax, capabilities, and use cases. Understanding both is crucial for Node.js development.

1) __CommonJS__

CommonJS was the original module system in Node.js. It uses synchronous loading and is widely supported across the Node.js ecosystem.

Syntax:

Exporting: Modules export their functionality using ```module.exports``` or simply ```exports```.
Importing: Modules are imported using the ```require``` function.

Example:

```javascript
// Exporting using module.exports
module.exports.add = (a, b) => a + b;
module.exports.subtract = (a, b) => a - b;
```

```javascript
// Importing using require
const math = require('./math');
console.log(math.add(2, 3)); // Output: 5
console.log(math.subtract(5, 3)); // Output: 2
```

Features:

* Synchronous Loading: Modules are loaded synchronously, which can block execution if loading takes time.
* Dynamic Imports: You can conditionally load modules using require statements anywhere in your code.
* Scope: Each file has its own module scope, preventing global variable pollution.

Usage: 

CommonJS is ideal for server-side applications where synchronous loading does not impact performance significantly and dynamic module loading is beneficial.

2) __ES Modules (preferred for modern projects)__

ES Modules (ESM) is the modern standard for JavaScript modules. It was introduced in ECMAScript 2015 (ES6) and is supported by modern JavaScript environments, including Node.js.

Syntax:

Exporting: Modules export functionality using ```export``` and ```export default```.
Importing: Modules are imported using ```import``` statement.

Example:

```javascript
// math.js
// Exporting using named exports
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
```

```javascript
// Importing using import statements
import { add, subtract } from './math.mjs';
console.log(add(2, 3)); // Output: 5
console.log(subtract(5, 3)); // Output: 2
```

Features:

* Static Analysis: The structure of ES modules can be analyzed at compile time, enabling optimizations like tree shaking.
* Asynchronous Loading: ES modules can be loaded asynchronously, which can improve performance by parallelizing the loading process.
* Syntax: The import and export syntax is more declarative and intuitive.
* Interoperability: ES modules can import CommonJS modules, although the reverse is not directly possible without workarounds.

Usage:

ES modules are ideal for both client-side and server-side applications. They align with modern JavaScript standards and are supported natively in browsers, promoting code reuse across different environments.

Example of Asynchronous Loading with ES Modules:

```javascript
// Importing dynamically using import()
import('./math.js').then(math => {
    console.log(math.add(2, 3)); // Output: 5
});
```

In the given context, import() is used as an asynchronous function. It loads and executes the 'math.js' module asynchronously, returning a promise. We can handle the loaded module using .then() after it has been successfully loaded.

__Configuring Node.js to work with ES Nodules__:

* Method 1: Use the .mjs extension for your JavaScript scripts. For example: ```script.mjs```.

* Method 2: Adding ```"type": "module"``` at the top level of your ```package.json```:

 ```json
 {
  "name": "your-project-name",
  "version": "1.0.0",
  "type": "module"
}
 ```
 
Ps1. Since method 2 requires changes in only one place, it is the preferred option.

Ps2. Choosing between CommonJS and ES Modules depends on compatibility, performance, and syntax preferences. CommonJS is widely supported in Node.js, making it the standard for legacy projects and many existing npm packages, but its synchronous loading can be a bottleneck. ES Modules, are preferred for new projects that require compatibility with both Node.js and browsers, offering asynchronous loading that improves performance through parallel module fetching and execution. 

### 6. Node.js core modules and usage examples

Node.js core modules are like ready-to-use tools that come with every Node.js installation. They offer essential functions for tasks like file system operations, creating HTTP servers, handling events, and more. These modules are included with Node.js, so you can use them right away without extra setup. 

The complete list of Node.js modules and their related documentation can be found here: https://nodejs.org/api/modules.html. To illustrate the usage of Node.js core modules, we have the following examples:

* __fs (File System)__: The __fs__ module enables reading, writing, and manipulating files and directories on the file system.

```javascript
import { writeFile, readFile } from 'fs/promises';

const content = 'Hello, Node.js!';

// Writing to a file
writeFile('example.txt', content)
  .then(() => console.log('File written successfully.'))
  .catch((err) => console.error(err));

// Reading from a file
readFile('example.txt', 'utf8')
  .then((data) => console.log(data))
  .catch((err) => console.error(err));
```

* __http/https (HTTP/HTTPS Servers)__: The __http__ and __https__ modules facilitate creating web servers and handling requests/responses.

```javascript
import http from 'http';

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello, Node.js HTTP Server!');
});

server.listen(3000, () => {
  console.log('HTTP server running on port 3000');
});
```

* __events (EventEmitter)__: The events module allows implementing event-driven architectures and custom event handling.

```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

myEmitter.on('customEvent', (arg) => {
  console.log('Custom event triggered:', arg);
});

myEmitter.emit('customEvent', 'Hello, Node.js Events!');
```

* __util (Utility Functions)__: The util module provides utility functions for various tasks such as object inspection, data conversion, and function inheritance.

Example (Promisify a Function):

```javascript
import { promisify } from 'util';
import { readFile } from 'fs';

const readFileAsync = promisify(readFile);

readFileAsync('example.txt', 'utf8')
  .then((data) => {
    console.log('File content:', data);
  })
  .catch((err) => {
    console.error('Error reading file:', err);
  });
```

* __path (File Path Handling)__: The path module deals with file and directory path manipulation, resolving and joining paths, and handling platform-specific path differences.

```javascript
import path from 'path';

const filePath = '/home/user/files/example.txt';
const absolutePath = path.resolve(filePath);

console.log('Absolute Path:', absolutePath);
```






