# Build Cross-Platform Apps Using Your Favorite Web Stack
### Reasonably Secure Edition

Hello and welcome everyone, thank you for joining us and taking the time to learn more about web-based cross-platform applications.

To set expectations from the start, I will explain what this talk will and will not cover. 

You won't learn about web development basics or frameworks, the Rust language or how to get your finished app into app stores.

Instead, we will cover the whole story of what you should know before and while building your Tauri app.

Tauri is a cross-platform app development framework and this talk will cover the basics of Tauri 2.0 and the development lifecycle with a focus on security considerations.

---
## What is an *App* really?

Before we get into technical details, I would like to redefine some basic term.

We all know the word app and probably use it at least once a week, but what is the generic definition of an app?

---
|----|
| > **Application software** |
| > **Mobile app**, software designed to run on smartphones and other mobile devices |
|> **Web application** or web app, software designed to run inside a web browser |


If you look at the Wikipedia page for "app", you will see different definitions of the word, depending on the context.

We have the very broad term Application Software, which describes applications in a more formal sense, but this is too abstract and not relevant for this context, as we are talking about practical applications.

The next two interpretations are easier for us to understand because we can see directly where they are being used.

Everyone here probably has a mobile device running Android or iOS, and most of you will find the phrase "app" most appropriate when talking about apps installed on your mobile device via an app store.

The last definition is the web app, which is something you interact with every day when you open your favourite website.

Traditionally, the last two definitions of apps are distinct and require different technology stacks, knowledge and have their own learning curves.

From a web developer's perspective, this makes it harder to develop for mobile platforms using native features such as the camera and sensors, as they have to learn a new technology.

Native mobile developers tend to focus on specific native frameworks that have different concepts than a web app, so it's also difficult for them to support both definitions.

This is expensive and time-consuming for companies, and not every developer is willing to change their tech stack mid-career.

---
#### Web 
#### Native
#### Simple
## Tauri

/assets/TAURI_Glyph_Color.svg
size: contain
x: right

This is where you can say hello to the new best friend of enterprises, web or native developers and anyone who supports a diverse user base on their respective platform.

The framework I am about to introduce is called Tauri.

Tauri combines the categories of desktop, mobile and web applications, allowing you to build an application for all major operating systems out there from a single code base.

It allows web developers to write cross-platform user interfaces the way they are used to, using their favourite web front-end framework, while interacting with the system.

It allows native developers to focus on great backend functionality for each platform and leave the writing of consistent user interfaces to web developers. They can also write backends as plugins, which means the native interaction can be reused by many different projects.

Before we get further into the technical side, I would like to demonstrate how easy it is to create a Tauri application from scratch.

---
/assets/bootstrap-project.mp4
size: contain

I installed the prerequisites of Tauri beforehand and this console recording shows the bootstrap commands using a cli tool called `create-tauri-app` provided from the Tauri website.

These are all commands needed to init the project and we are ready to start developing a new application for desktop and mobile.

After completion of the init process and installing the frontend packages I ran `pnpm tauri dev` and `pnpm tauri ios dev` to run both versions in debug mode on my machine.

---
/assets/hot-reload-1080.mp4
size: contain

You can see the code change I make in the `index.html` on the left live updates both Tauri apps at the same time.

This is called Hot Module Reload (HMR). This means that any changes made to the front-end code in my editor are automatically displayed side-by-side on both platforms, without recompiling the backend code.

This is achieved using a local dev server and this also makes it possible to inspect and debug the frontend code in your local web browser.

In release builds, this feature is disabled and no port or server is exposed to the system.

Technically, you can run remote or local emulators and develop with this Live Preview for all platforms at the same time.

Tauri theoretically runs on many more platforms depending on WebView and Rust target support, so we have people experimenting with visionOS, watchOS, androidTV and even game consoles.

---
## The Tech Stack
/assets/TAURI_Glyph_Color.svg
size: contain
/assets/WRY_Logo_Dark.svg
size: contain
/assets/TAO_Logo_Dark.svg
size: contain

/assets/rust-logo-blk.svg
size: contain
/assets/Kotlin_logo_2021.svg
size: contain
/assets/Swift_logo.svg
size: contain

/assets/WebKit_logo_(2015).svg
size: contain
/assets/Chromium_Logo.svg
size: contain

So let's have a look under the hood.

Tauri is primarily built using Rust, a powerful low-level programming language with a focus on security. 

Rust is said to have a steep learning curve compared to JavaScript or other high-level languages.
This is probably the biggest barrier for new developers, but for our focus on security, performance and binary size we had no real alternatives.

Rust is type-safe and has a very strict compiler, which enforces good engineering habits and prevents us from taking shortcuts in many places.

Some core concepts of Rust prevent classic memory vulnerabilities of low-level languages such as C/C++.

Tauri supports mobile operating systems, and to achieve this it needs a way to interact with the native mobile system APIs.

We considered staying Rust only, but this would have required us to build and maintain API bindings for each operating system. That would have been complex and tedious.

We try to pick the right tool for the job. This resulted in a small amount of wrapper code written in Kotlin and Swift that simplifies system API access from Rust.

Tauri combines two underlying libraries, also written in Rust.

Tao, which is responsible for handling, creating and interacting with native windows.

Wry, which is responsible for initialising and interacting with the WebViews used on the operating system.

Tauri uses WebViews to render and display the frontend code.
These are different on each operating system, and we will discuss later why this is actually an advantage in most scenarios.

On Windows, Tauri uses Webview2, which is based on Chromium. 
On Android it uses another Chromium-based WebView called Blink.

On Linux, it makes use of WebKitGTK, which is based on Webkit, which is used on MacOS and iOS.

For frontend developers, mentioning these WebView stacks means that you can use any frontend framework that works with these WebViews. We have templates for popular frameworks like React, Vue, Next.js, and even Rust based WebAssembly frontends like Yew work out of the box.

---
## WebViews
	To Bundle, or Not to Bundle?

Talking about WebViews brings us to an important question. To bundle or not to bundle?

The topic of directly bundling a WebView with application code that interacts with the native operating system is not new and has been used by a fair number of frameworks that have been around for a while.

At the beginning of Tauri this question was part of very important discussions.

A common application with a bundled WebView would easily exceed 100 megabytes with almost no added functionality.

When playing the numbers game by simply calculating the traffic, disk space, environmental overhead and distribution latency for shipping a simple application to thousands of users, it was clear that relying on either pre-installed WebViews or providing an installer to install these WebViews on a system would be the more sustainable choice.

Another very important aspect from a security perspective influenced this decision and that is the security update lag.

---
/assets/tauri_update_lag.svg
size: contain

The issue of update lag is all about responsibility.

When bundling a WebView into your application, you as the developer are responsible for ensuring that the latest WebView security patches are included in your latest release.

This means that for each security update of the WebView you need to download or compile the latest available version, bundle it with your application code and then ensure that the latest version is distributed to your user base.

In practice, very few applications get this right and automate it well enough to keep up with the pace of WebView security updates.

For Tauri applications it is a different story, where you simply ship your application code and rely on the WebView being properly updated by the package manager or directly by the operating system.

In this case, the package or OS maintainer is responsible for keeping the WebView versions up to date.
In most cases, this process greatly reduces the time it takes to patch.

---
	Web vs Native
## Threat Model

The threats to a web application running in a browser are very different from the threats to a native application running directly on a system.

When you build a web application, the code is usually running on a web server that you own or at least control. When you run the application in a browser, you don't have to worry about other tabs, windows or things on the user's system.

The browser sandbox and environment will take care of that for you.

You can assume that your application won't be able to compromise the end user's device, so you only have to worry about risks from your application logic and, of course, your own bugs.

When building a native application, there is no browser sandbox or environment and you are responsible for writing code that does not compromise and break the user's device.

For the threat model of a Tauri application we need to consider a mix of both. But to fully understand where to apply which perspective, we need to understand the trust boundaries.

---
## What is a Trust Boundary?

	> Trust boundary is a term used in computer science and security which describes a boundary where program data or execution changes its level of "trust," or where two principals with different capabilities exchange data or commands. (*Wikipedia*)

Trust is something depending on the implementation and the perspective. It is a very broad term and is a central topic in IT-Security and the exact definition needs to be created for each scenario.

Another thing we can derive from the definition on the slide is that there are different levels of trust and sometimes data is passed between these levels. When something is transferred into another level, it can end up in a higher or of lower level of trust.

Whenever data changes it's level, it has to pass a boundary and this is where the uncertainty happens and actual security vulnerabilities can occur.

Whenever something passes this boundary unintentional it is called a trust boundary violation.

Inspecting and strongly defining all data passed between boundaries is very important to prevent these trust boundary violations. If data is passed without access control between these boundaries then it's easy for attackers to elevate and abuse privileges.

---
## Trust Boundaries
/assets/tauri_trust_boundaries.svg
size: contain

A Tauri application consists of two main trust levels, the WebView maintained by the local system running the front-end code, and the native code of the Tauri application.

The WebView has access to the network but is isolated from full system access. Code written for the front-end must take into account the typical web threat model and, in Tauri's case, also be aware of exposed system features. We don't need to trust this code beyond the exposed system features, if we exclude 0-day exploits in WebView code.

We call the native part the application core. It contains Tauri's core code, the application-specific backend code and the application plugins. Most of these components are written in Rust, but sometimes they use or interact with other native languages and bindings.

The native component has full access to the local system without any restrictions imposed by the Tauri framework. Realistically, there is no way for us to create a fully isolated runtime without spending most of our time building and maintaining it.

This means that we have to start with a certain level of trust. We trust application and plugin developers to write non-malicious Rust code.

As I said, Rust itself enforces some good habits, but these alone won't make your application reasonably secure.

We will talk later about what can still go wrong and what you can do to reduce the risks.

First, we'll focus on the WebView and how it communicates with the application core.

---
## IPC
	Communication across Trust Boundaries

Tauri uses something called Inter Process Communication, or IPC, between the core and WebView components.

The frontend has no direct system access by default and all requests to access resources outside the WebView process must go through a well-defined communication protocol.

This communication takes place between the WebView process and the Tauri process, which means that there is no system wide exposure through a port or socket.

Tauri uses a particular style of Inter-Process Communication called Asynchronous Message Passing, where processes exchange requests and responses serialized using some simple data representation.

---
## Tauri Commands
### Rust Backend
```rust
#[tauri::command]
fn my_greeting(message: String) -> String {
  format!("{message} from Rust!")
}
```
### JavaScript Frontend
```js
invoke('my_greeting'{ message: 'Hello!' }).then((greeting) => console.log(greeting))
```

To interact with the Rust based application core, Tauri has a concept called a command. This allows you to implement heavy computation or IO access logic in Rust. It is very easy to use and is used to expose system functionality to the frontend.

Tauri itself and existing Tauri plugins expose commonly requested features such as file system access using this command implementation.

For the Rust aware audience: The returned data can be of any type as long as it implements `serde::Serialize`.

For front-end developers, many plugins already expose generated Typescript bindings to their commands, so you can stay in your Intellisense comfort zone and write maintainable code.

---
## Recap
	Getting Beyond Basics

We've discussed the very basic Tauri knowledge up til now. A very simplified, too long didn't listen recap for the next few slides:

Tauri applications are local applications that run on the end user device, mobile or desktop, contain backends written primarily in Rust, frontend written in any web framework, communication exists between both components.

As mentioned earlier, web developers tend to live in a happy bubble, confined to a browser, with backend code running on a trusted server.

---
## Hostile Environment
	You can't trust anything!

The reality of local native applications is not so friendly and there is no trust in the local device executing the backend logic. So if you embed secrets in your app, you're really screwed: other apps will try to steal your data, users will reverse-engineer your code, you can't even trust the outcome of cryptographic operations because the system could always be manipulated.

This is scary from an app developer's perspective, but also from a user's perspective.
Who wants applications to be able to go havoc on their system?

So this end in to two major questions:

- How can the user restrict apps to access only what they should?
- How can app developers build their applications to regain some kind of trust from users?

---
## Sandboxing
	Finding The Balance

These old questions have led to an innovation in technology known as sandboxing.

Sandboxing allows developers, users and operating systems to define access controls to resources. The main problem with sandboxing is perfectly described in this [xkcd](https://xkcd.com/2044/).

---
/assets/sandboxing_cycle_2x.png "https://xkcd.com/2044/"
size: contain

Whenever you restrict something for security reasons, people feel restricted. They want something simpler and easier to use to get quick results.

They want something more communicative and easy to use, with no configuration required.

Then someone builds this easy-to-use open system, which then takes everyone who uses it back to square one, where sensitive data or access is overly shared again.

Security researchers will exploit this and come up with new isolation features to prevent abuse.

People will feel restricted again and look for simpler solutions.

This cycle repeats itself across all your devices and software frameworks.

---
## Container
	Just Put It in a Box

A common approach that most of you are probably familiar with is a container. This feature requires the operating system to integrate and provide privileged tools to isolate user level applications based on rules.

Popular desktop tools and frameworks include Docker, Flatpak and Snap. These solutions require either the developer to provide a runtime or instructions and configurations for the user to make it work.

There are built-in solutions such as the iOS and Android app isolation and permission system. Here the developer must provide manifests describing the desired system access and the user can grant or revoke this at runtime.

This approach can help to restrict system access after the application has been shipped, but it is usually not very fine-grained and it is difficult to find a balance between restrictions and usability.

Tauri is compatible with this container sandboxing approach and has active effort to make configuration easier.

---
## Application Logic Sandbox
	Reasonable Applications

A different approach is to consider the privileges and access requirements of an application at compile time. This requires developers to fully understand what access they need when they build their application. 

This can be very fine-grained, with a perfect balance between access restrictions and usability, as it is considered early in the development process.

This approach does not protect against other hostile applications on a system. Rather, it is designed to provide a level of trust between the developer and the user, while reducing the impact of security flaws within the application.

Tauri has built-in application level sandboxing features called capabilities, permissions and scopes. These are designed to control the system access of the Frontend.

Let's explore Tauri's built-in sandboxing capabilities.

---
## Permissions
	Define Command Exposure
```toml
[[permission]]
identifier = "my-permission"
description = "Reading files is only exposed on Windows"
platforms = ["Windows"]
commands.allow = [
    "fs:read_file"
]
```

Permissions are descriptions of the explicit permissions of Tauri commands.

We have already discussed what a Tauri command is. What we did not discuss there is that plugins can have a lot of different functionality and not every application using the plugin needs all the exposed commands.

So to reduce this over-exposure, the application developer can configure which commands are exposed in their application at compile time.

Plugin developers can provide out-of-the-box permissions that enable single commands or a set of commonly used commands.

Sometimes this exposure only makes sense on certain platforms, so it is possible to list the platforms to which a particular assumption-based permission applies.

This example allows the `read_file` command to be exposed on Windows builds only. It uses the `fs` filesystem plugin, which is indicated by the namespace in front of the command name.

Permissions can also be used to fine-tune access to enabled commands by using something called scopes.

---
## Scopes
	Define Fine Grained Access
```toml
[[scope.allow]]
path = "$HOME/*"

[[scope.deny]]
path = "$HOME/secret"
```

Scopes allow you to extend permissions so that the application or plugin developer can go beyond simply enabling or disabling commands.

This example is an extension of the previous permission, and allows our application to read the contents of the home folder, while preventing access to the secret folder.

The scope type is defined by the plugin developer and can be of any serialisable type. Of course, application developers can extend or define their own scope values.

In this example we are configuring the `fs` scope type `path`, which is a string containing `glob` pattern paths.

Each command can either define it's own scope type or use one type for the whole plugin.

The important thing to note here is that even if the configured scope is correct and safe, the command developer must verify the scope correctly.

This means that it is very important that plugin developers get this component right and do proper security checks.

Since Tauri does not know what your commands implement, we can only provide mechanisms that make it easy to build proper access control.

Ultimately, it will run on the user's device with your application code.

---
## Capabilities
```json
{
  "identifier": "mobile-capability",
  "windows": ["main"],
  "platforms": ["iOS", "android"],
  "permissions": [
    "nfc:allow-scan",
    "biometric:allow-authenticate",
    "barcode-scanner:allow-scan"
  ]
}
```

The final component of the access control system in Tauri is the capability.

A capability attaches permissions to windows or WebViews.

Tauri applications can run with multiple windows or WebViews. These can host different content with different security assumptions, so each window or WebView can use different permissions.

This is what application developers need to configure and where they rely on plugin or backend developers to have scoping properly implemented.

Tauri itself enforces the exposure of commands and ensures that the correct scope is passed to the command itself.

This example exposes the NFC functionality to scan tags with `nfc:allow-scan`, the biometric authentication by enabling `biometric:allow-authenticate` and the barcode scanning functionality with `barcode-scanner:allow-scan`.

All these commands are only exposed on iOS and Android and only to the `main` window.

The capabilities also allow to define if a remote website is loaded in the WebView and should get access to the Tauri APIS. This is quite dangerous, so we don't allow it by default and it must be explicitly configured.

As the application developer, you should know best what the threat model is for each window and the commands used.

The complete model of permissions, scopes and capabilities is designed to streamline and simplify access control by providing a common interface for application and plugin developers.

This can be extended at runtime, depending on plugin support.
---
## Isolation Pattern
	Frontend Developer's Hidden Power

Another approach to restrict and control the exposure of the Tauri core to the frontend can be handled by the frontend developer themselves, using another built-in feature of Tauri.

This feature can be easily enabled in the configuration and injects a top level iFrame into the web application and all application frontend code is run in a sub iFrame.
We call this approach "Isolation Pattern".

This means that all IPC communication first passes through the isolation iFrame via `postMessage`.
This iFrame can be used to inspect, modify or block the communication by writing this logic in JavaScript.

We can imagine developers writing their own web application firewall (WAF) or using it to detect compromise.

We haven't seen massive real-world adoption of this feature, so there are probably not enough resources and use cases for most front-end developers.
We imagine this pattern will be helpful for Tauri application developers who are not familiar with Rust and need to implement custom security checks.

---
## Content Security Policy
	First Layer of Defense
```text
default-src 'self'; connect-src ipc: http://ipc.localhost
```

Tauri applications support the well known Content Security Policy (CSP) based on the WebView system.

This policy should be part of the basic knowledge of a web developer, but if you are not yet familiar with it, I can recommend the [Mozilla documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP).

In Tauri applications, this policy is usually the first layer of defence, reducing the impact of common web vulnerabilities such as cross-site scripting and data leakage.

All bundled assets in a Tauri application are automatically hashed and nonce'd during the build process and then appended to the CSP. 
This means you can focus on restricting allowed remote sources.

---
## Application Development Lifecycle Threats
	The Weakest Link Defines Security

/assets/security.png "https://xkcd.com/538"
size: contain

We have covered the basic security features of Tauri, sandboxing and security boundaries.

With this knowledge, you would be able to write a fairly hardened application that could pass as reasonably secure.

We could conclude the talk here and going home you would likely be thinking:

> "Oh nice I learned a bit about Tauri and security, now I am writing super secure Tauri applications".

But the weakest link in your application lifecycle essentially defines your security, so we need to cover more to actually get to a reasonable security baseline.

I will quote from the Tauri security documentation.

> It is important to remember that the security of your Tauri application is the sum of the overall security of Tauri itself, all Rust and NPM dependencies, your code, and the devices that build and run the final application.

> The Tauri team does its best to do its part, the security community does its part, and you should also follow some important best practices.

These best practices apply to several steps in the application development process.

Each step we discuss from now on can compromise the assumptions and integrity of all subsequent steps, so it is important to see the whole picture at all times.

---
## Upstream
	The Fragile Shoulders You Stand on

/assets/dependency.png "[https://xkcd.com/2347](https://xkcd.com/2347)"



Let us start with everything you consume before you can even compile a Tauri application.

Upstream is a very broad term and we won't be able to cover everything relevant in these slides and will mainly focus on Tauri related dependencies.

Tauri applications depend on WebViews, which means they need to be installed in order to develop and run applications. You have no real control over this, so keeping track of security announcements and pushing them out to your users is really all you can do with reasonable effort.

Tauri itself consumes at least the Rust ecosystem and will introduce hundreds of crates as direct or transient dependencies.

Have you ever used a Rust crate and executed `cargo tree`?
Have you ever run `npm -ls --all` on your frontend project?

There is a reason why people develop graphical tools to visualise and understand dependencies. It's just too much to keep track of.

All those packages need to be updated in case of security patches, and all of them could be a security risk that compromises your whole chain before you have written a single line of code.

Supply chain attacks are a common threat in modern application stacks, and there are best practices for choosing your dependencies, but frankly, none of the following practices will be enough if a state actor decides to compromise you in this way.

---
## Upstream Rust
	`cargo audit`, `cargo auditable`, `cargo vet`, `cargo crev` & `cargo-supply-chain`

Let's focus on the Rust ecosystem for now.

How do you make sure you have the latest and patched cargo packages in your project?

This is where `cargo audit` comes in. It automatically fetches and displays information about packages with known vulnerabilities and compares them with your configured dependencies. Not all vulnerabilities apply directly to your project, so you will need to manually assess the impact.

The complementary tool for this is `cargo auditable`, which embeds all dependencies and their versions as readable data in your compiled rust application.

This allows you to use `cargo audit` on applications deployed in the wild, making it easy to assess risk and generate SBOMs in production. 

But what if there is a new version of a package you use with a malicious change that compromises your system?

A Recent example would be the `xz` package which had a maintainer implementing a backdoor to take over Debian machines remotely.

We can't expect developers to read the source code of all their dependencies for every update - it's just not feasible for complex applications.

This is where tools like `cargo crev` and `cargo vet` shine.

Their common goal is to provide a way to trust dependencies, where the responsibility for vetting crate versions is distributed among trusted peers. 

This allows developers to run trusted dependency versions without having to check every crate themselves.

Both have different nuanced approaches to achieving this goal, and you should evaluate which approach makes sense in your threat model.

To find out on whose shoulders you stand, you can use the `cargo supply chain` tool.

It shows you every org and individual being able to publish a dependency you are using.
This knowledge can be used as a reality check and to find out who you should be supporting.

---
## Upstream JavaScript
	`npm audit` 
	That's it?

We have at least one more ecosystem to worry about. The JavaScript ecosystem, with its common `npm` packages, is at least as complex as the Rust crate system, but has even fewer integrated projects to address supply chain issues.

There are commercial and free services like `snyk` or `socket.dev` that will analyse your dependencies and alert you when dependency attacks are detected, but they are not a general recommendation for every project and you should find out which service suits your application.

The only generic tool that you are probably familiar with is `npm audit`, which is similar to `cargo audit` and highlights known vulnerabilities in your dependencies.

There are several guides to hardening npm, relying on lock files, preventing typo squatting, auditing build scripts, but all of these require the developer to apply them in a correct and nuanced way, so I encourage you to do your own research and apply them to your project if necessary.

---
## Development
	It's Your Responsibility

Let's say you've done everything right with your project's upstream dependencies, and now you want to develop and debug your application.

To do this, you need a device running the operating system you want to support with your application. Hardening your operating system is of course necessary, but is too complex to cover in this talk.

You can of course virtualise your development environment to keep attackers at bay, but this won't protect you from attacks that target your project rather than just your machine.

---
## IDEs
	Code Execution Everywhere

You will usually use an integrated development environment (IDE) to edit and interact with source code. These editors all have different threat models and you need to consider what you trust.

Let's look at some examples of what can happen, when using Visual Studio Code.
Most of these apply to other IDEs as well, but VSCode is the most prominent.

What happens when you check out and open a foreign repository?

---
/assets/vscode-rce.mp4

VSCode will ask you if you trust it, because it has so many features that a malicious repo could execute code on your system, and they can't guarantee that this won't happen with the default features enabled.

This is why we see code executed in the terminal and a meme displayed in the embedded browser feature without running anything.

Let's be realistic - who here **always** opens unknown repositories in the restricted mode?
The user interface suggests to go with the nice blue trust button anyway.

VSCode has introduced a sandbox for untrusted projects to limit features, but even this sandbox is not fail-safe, so it is really necessary to trust projects before opening them in VSCode.

This is a paradox as you are just starting with a previously unknown project and want to inspect it before trusting it.

Opening untrusted files is always risky, so you may want to use a container, virtual machine or a completely fresh system to inspect these repositories first, but I can't give you the perfect nuanced solution without considering your entire setup.

What about compiling a foreign project?

---
Suppose the following rust `main.rs`:

### `main.rs`
```rust
fn main() {
    // Super safe to run me!
    println!("Hello, world!");
}
```
```
Hello, world!
```
Assume you just checked the `main.rs` file in the untrusted project and concluded it's safe to compile.

You would expect the output below right?

But then you find something weird after compiling the foreign project.

---
## PWNED?
```
warning: rust-build-demo@0.1.0: PWNED
Hello, world!
```

When you compile Rust projects, attackers can execute code via the `build.rs` system at compile time, so it's necessary to check your project's `build.rs` files before compilation, unless you explicitly trust the project. Additionally, all of your dependencies `build.rs` files will get executed. Inspecting these is a tedious task very few people actually do, as mentioned during the upstream topic.

---
### `build.rs`
```rust
fn main() {
    println!("cargo:warning=PWNED");
    // go wild here
}
```

The `build.rs` can execute arbitrary code and this example code is just nice in showing a compiler warning.
The same applies to `npm` with pre- and post-build hooks.

There are a lot of other ways to compromise your development system via exploits or unknown behavior of your IDEs.

You can containerise your IDE to reduce system impact, but you can't reasonably protect your code from a compromised IDE.

---
## Secrets
	 A Developers Nightmare
	`dev.env` `prod.env` `idontknow.env`

Managing secrets during development is difficult and most people tend to be blind on this topic as long as they haven't been compromised before. 

A common way is to store git access and signing keys in hardware tokens, but this only protects secrets from being exposed, not from being accessed by compromised development machines.

For developer secrets that can't be stored on a hardware token, you need to make sure that these only have access to development environments.

Also committing secrets is a thing. There are still thousands of secrets in public git repositories.
Sometimes not only development secrets but also production secrets.

There are automated secret scanning services integrated into GitHub, GitLab which should be enabled for public repositories but it's better to not completely rely on these.

A good setup has proper push protection for secrets and manages secrets via secret managers. 

Advice I can't stress enough: Never use or store production secrets on development machines.

---
## Build
	Trust, Trust, Trust?!

So you have made the long journey to finally build a release version of your application.

The first question is where to build it?
Locally or remotely? 

In most cases it will be remote, because you don't want to build on your development system, and you don't have a dedicated release building system.

Sometimes you explicitly want to build releases locally, because you don't want to trust others with your production secrets. This allows you to use hardware under your control, which simplifies the threat model, but makes cutting releases a bus factor and complicated in fast-paced development.

So let's focus on remote build systems, which are more realistic and complicated.

When you use a platform like GitHub to compile and produce release assets, you trust them to keep your production secrets safe, that their runners (build machines) are not compromised, and that the result of the release is really just your code and no sneaky backdoor has been added.

---
## Reproducible
	The Same As Always, Please!

To combat backdoor injection at build time, you need your builds to be reproducible, so that you can verify that the build assets are exactly the same when you build them locally or on another independent provider.

The first problem is that Rust is by default not fully capable of reliably producing reproducible builds. It supports this in theory, but there are still bugs, and it recently broke on a release. You can keep track of the current state in the rust project's public bug tracker.

The next problem you will encounter is that many common frontend bundlers do not produce reproducible output either, so the bundled assets may also break reproducible builds.

This means that you cannot fully rely on reproducible builds by default, and sadly need to fully trust external vendors to build your application.

---
## Production Secrets
	```yml
	# Keep This Secret!!!
	PROD_SECRET="Correct Horse Battery Staple"
	```

Production secrets are a tricky thing. You __MUST__ trust a remote entity not to abuse or leak your secrets. 

Tauri has an updater plugin for desktop builds that allows the application to update itself. The private key used to sign the updater assets must not be leaked under any circumstances, as the whole process depends on it being a secret value. These are exposed at compile time, so as said before you need to trust the build system.

If cryptographic secrets are properly stored on hardware tokens, a compromised build system won't be able to leak involved signing keys, but could use them to sign malicious releases.

Other secrets used in your build flow that are not stored on hardware tokens are fully accessible to your build systems and should therefore be rotated regularly and monitored for compromise.

---
/assets/cloud-icon.svg
size: contain
	[Crabnebula.Cloud](https://crabnebula.cloud)
## Distribute

You have successfully built, signed and validated your application, and now you want to distribute it to your users.

This is the last step you can at least partially control before it gets into the hands of fully untrusted systems with users eager to hack your application.

For initial distribution, and depending on the operating system you are targeting, you can distribute via app stores or direct downloads from your website.

There is a new solution for making distribution of Tauri apps easy and convenient at [crabnebula.cloud](https://crabnebula.cloud) which is linked here. Currently offering free trials and open source plans. Disclaimer: I work for this company and am involved in the security of the product.

The infrastructure for hosting your Tauri updater endpoint shouldn't be compromised, but the biggest impact, apart from simply not shipping the update, would be a downgrade attack by shipping an older and vulnerable version that pretends to be a newer version.

So some trust is needed in the distribution infrastructure, but a lot less than in the build and signing infrastructure.

You should publish the signatures/hashes of your application build assets so that users can verify them locally.

---
## Runtime
	It's Too Late Now, But it's Okay

So your application has made it to the user, what can go wrong at this point?

Anything, because it's the most hostile environment imaginable, but there's almost nothing you can do about it.

Giving your users an easy and secure way to report vulnerabilities and keeping a close eye on your project, is the last thing you can do.

You will always have a weak link in your chain, but as long as you have properly implemented and thoroughly considered everything we have discussed so far, you should have a reasonably secure cross-platform application to be proud of.
 

---
## Tauri
/assets/TAURI_Glyph_Color.svg
size: contain
	Security Team
	[tillmann@tauri.app](mailto:tillmann@tauri.app)
/assets/qrcode-20240307-045705.svg
size: contain

## CrabNebula
/assets/Icon.svg
size: contain
	Director of Security
	[tillmann@crabnebula.dev](mailto:tillmann@crabnebula.dev)
/assets/qrcode-20240307-045712.svg
size: contain

All of the presented Topics are just a general introduction, so you can go way beyond the mentioned steps and there is still a lot missing for more nuanced threat modeling but most of these topics deserve an individual talk. 

If you have any questions or would like to discuss some topics more in depth, feel free to reach out. Either in the Tauri project or to my employer CrabNebula, making this presentation and a lot of my contributions around Tauri possible.

Lastly, I have compiled a non-exhaustive collection of previous work I can recommend and will show after this slide.

Thank you for your attention and time, it was a pleasure.

---

/assets/talk-hardening-open-source.svg
size: contain
#### [Talk - Hardening Open Source Development](https://media.ccc.de/v/34c3-9249-hardening_open_source_development)

/assets/qrcode-20240307-043221.svg
size: contain
#### [Talk - Reproducible Builds, the first ten years](https://media.ccc.de/v/camp2023-57236-reproducible_builds_the_first_ten_years)

/assets/qrcode-20240307-043411.svg
size: contain
#### [Talk - SLSA, SigStore, SBOM & Software Supply Chain Security. What does it all mean?](https://www.youtube.com/watch?v=hF95PiItWtM)

/assets/qrcode-20240307-043441.svg
size: contain
#### [Video - What are hardware security modules (HSM), why we need them and how they work.](https://www.youtube.com/watch?v=szagwwSLbXo)

/assets/qrcode-20240307-043559.svg
size: contain
#### [Video - What is a Browser Security Sandbox?!](https://www.youtube.com/watch?v=StQ_6juJlZY) 

/assets/qrcode-20240307-043254.svg
size: contain
#### [Guide - OSSF NPM best practices Guide](https://github.com/ossf/package-manager-best-practices/blob/main/published/npm.md)







