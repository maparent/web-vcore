# web-vcore

Core code shared between my projects, for web-app frontends.

### Installation

> todo: update these instructions

1) Install the package: (or symlink it; that's recommended for long-term usage)
```
npm install web-vcore
```
2) [opt] Add the following to your code entry-file (or anywhere really):
```
import type {} from "web-vcore/nm/@All"; // helps vscode's auto-importer notice the "web-vcore/nm/*" package re-exports
```
3) Add the following to your `tsconfig.json`: (these entries are only needed for packages that are peer-dependencies of a user-project node-module)
```
"compilerOptions" {
	"paths": {
		"react": [
			"node_modules/web-vcore/node_modules/react",
			"node_modules/react",
		]
		// for monorepo:
		/*"react": [
			"../../../node_modules/web-vcore/node_modules/react",
			"../../../node_modules/react",
		]*/
	},
},
"references": [
	{"path": "node_modules/web-vcore"},
	// for monorepo:
	//{"path": "../../node_modules/web-vcore"},
]
```
4) Make-so web-vcore's package-patches get applied, by adding/merging the following command into your project's `postinstall` script:
```
"postinstall": "cd ./node_modules/web-vcore && node ./Scripts_Dist/ApplyPatches.js"
```
5) Various other things, like populating the RootStore interface. (for now, just reference an existing project that uses web-vcore, as seen below)

### Creating package patches

Regular: `patch-package patch MY_PACKAGE`
With package.json: `patch-package patch MY_PACKAGE --exclude 'nothing'`

If including package.json, modify the diff file afterward to omit the npm-install-related noise.

For details on how the patch files are parsed, see here: https://github.com/ds300/patch-package/blob/5c2c92bf504885fba4840870a23fc8999c00e572/src/patch/parse.ts

### Usage of yalc/zalc with subdeps

> todo: ensure these instructions are up-to-date

Steps to newly-link subdep:
* 1\) Run `zalc push` in subdep's source repo.
* 2\) Run `zalc add SUBDEP_NAME` in web-vcore repo. (without `--pure`, as that doesn't create symlink in `node_modules`)
* 3\) Open `yalc.lock`, remove the fields other than `signature` for the subdep, then add the field `pure: true`.
* 4\) Open `package.json`, and set the version for that subdep (back to) the current version number. (must update this when version changes in subdep's source repo)
* 5\) Run `yarn` in web-vcore repo.
* 6\) Run `zalc push` in web-vcore repo.
* 7\) Run `yarn` in user project.
* Note: If you need to repair the linkage manually fsr, use a tool like [Hard Link Shell Extension](https://schinagl.priv.at/nt/hardlinkshellext/linkshellextension.html) to create a Junction from `node_modules/X` to `.yalc/X`.

### Documentation

For the most part, web-vcore is meant to be learned/used based on referencing the codebase of existing projects using it (eg. [Debate Map](https://github.com/debate-map/app)). This is because the package is only intended/expected to be used for projects I'm building, so there's not that much benefit to creating full-fledged documentation.

That said, there are some things (eg. particularly complex components) for which documentation is worth creating, both for collaborators on projects I've made, as well as for my own reference in the future.

* [Abbreviations and Comments](./Docs/AbbreviationsAndComments.md)
* [Database Migrations](./Docs/DatabaseMigrations.md)

### What is the purpose of web-vcore? And why is web-vcore updated so frequently?

There are two primary functions of the web-vcore package:
1) Serve as a "container of subdependencies", such that by adding web-vcore to a project, you get a set of subdependencies with it that are known to work well together, with consistent versions that are used across my web projects.
2) Because my web projects use a lot of custom libraries, and because I frequently make small "micro changes" to those libraries, I've found it helpful to combine web-vcore with use of [yalc](https://github.com/wclr/yalc) (or in some cases, a fork of it called [zalc](https://github.com/Venryx/zalc)). Specifically, I use yalc as a way to have local changes made to custom libraries (ie. subdependencies of web-vcore that I've written myself) propagated up to the higher-level web projects, without needing to constantly be making npm publishes of both the subdependencies and the web-vcore container package; I yalc-push from the subdep into the web-vcore package, then yalc-push from there into the larger web project(s).

Due to local library changes being propagated to my web projects without needing npm-publishes, this of course raises the concern of me neglecting to actually push those changes to a place where other developers of my web projects can access them. The middle-ground I've settled on for now is:
* Include the `.yalc` folder of web-vcore in the contents that npm publishes for the web-vcore package. Thus, as long as I remember to do an npm-publish of the web-vcore package each time I make subdependency changes, I don't need to bother about comprehensively tracking down what subdependencies were changed during each web-project commit, in order to make additional library publishes. (I do of course eventually make npm publishes for those subdependencies, but this way I can do so at convenient times and with greater care.)

While this approach is non-standard, and not perfect, it has been very helpful from a practical standpoint, in terms of letting me iterate quickly while still maintaining the separation of logic into lots of small components/libraries.