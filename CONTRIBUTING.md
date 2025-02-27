# Contributing

Contributions to KeystoneJS in the form of issues and PRs are welcomed.

During the alpha stage of development we are focussing on getting the core systems working smoothly.
Contributions which improve the documentation and test coverage are particularly welcomed.

### Community Ecosystem

Keystone makes no assumptions about type of applications it powers. It achieves flexibility through small, highly composable parts that allow you to build a foundation for any type of application.

For this reason we might not add features to Keystone if they are prescriptive about:

- Data structures
- Workflows
- Access controls
- Front-end application UI

But we want your contributions! We recognise many types of applications share common features and prescriptive patterns can sometimes be helpful, even at the expense of flexibility.

If you develop custom fields, adapters, apps or any other Keystone feature, (or have an idea) join us on the [Keystone Slack channel](https://launchpass.com/keystonejs) or make a pull request to [KeystoneJS-Contrib](https://github.com/keystonejs-contrib/keystonejs-contrib) and we will add it to our list of community libraries.

## Code of Conduct

KeystoneJS adheres to the [Contributor Covenant Code of Conduct](code-of-conduct.md).

## Repository Setup

KeystoneJS follows the [Thinkmill Monorepo Style Guide](https://github.com/Thinkmill/monorepo). For more information on the reasoning behind using certain tooling, please refer to it.

### Version management

Keystone uses @noviny's [@changesets/cli](https://github.com/noviny/changesets) to track package versions and publish packages.
This tool allows each PR to indicate which packages need a version bump along with a changelog snippet.
This information is then collated when performing a release to update package versions and `CHANGELOG.md` files.

#### What all contributors need to do

- Make your changes (as per usual)
- Before you make a Pull Request, run the `yarn changeset` command and answer the questions that are asked. It will want to know:
  - which packages you want to publish
  - what version you are releasing them at
  - a message to summarise the changes (this message will be written to the changelog of bumped packages)
- Before you accept the changeset, it will display all the data that will be written to the changeset. If this looks fine, agree, and a changeset will be generated in the `.changeset` directory.

After this, a new changeset will be added which is a markdown file with YAML front matter.

```
-| .changeset/
-|-| UNIQUE_ID.md
```

The message you typed can be found in the markdown file. If you want to expand on it, you can write as much markdown as you want, which will all be added to the changelog on publish. If you want to add more packages or change the bump types of any packages, that's also fine.

While not every changeset is going to need a huge amount of detail, a good idea of what should be in a changeset is:

- WHAT the change is
- WHY the change was made
- HOW a consumer should update their code

An example, if you generate a changeset that includes `adapter-mongoose` as a patch, and `keystone` as a minor, you can merge your PR, and the next time the `version-packages` command is run, these will both be updated.

```md
---
'@keystone-alpha/adapter-mongoose': patch
'@keystone-alpha/keystone': minor
---

A very useful description of the changes should be here.
```

You can have multiple changesets in a single PR. This will give you more granular changelogs, and is encouraged.

### Release Guidelines

#### How to do a release

> This should only ever be done by a very short list of core contributors

Releasing is a two-step process. The first step updates the packages, and the second step publishes updated packages to npm.

##### Update Packages (automatic)

This step is handled for us by the Changesets GitHub Action. As PRs are opened
against `master`, the bot will open and update a PR which generates the
appropriate `CHANGELOG.md` entries and `package.json` version bumps.

Once ready for a release, merge the bot's PR into `master`.

> _NOTE: For information on manually updating packages, see [Update Packages
> (manual)](#update-packages-manual)_

##### Publish Packages

Once the version changes are merged back in to master, to do a manual release:

```sh
yarn fresh && \
yarn publish-changed && \
git push --tags
```

The `yarn publish-changed` command finds packages where the version listed in the `package.json` is ahead of the version published on npm, and attempts to publish just those packages.

NOTE: There is no reason you should ever manually edit the version in the `package.json`

##### Update Packages (manual)

If you wish to do a manual release (useful for back-porting fixes), follow these
steps. Otherwise, skip on to the next section for _Publishing Packages_.

The first step is `yarn version-packages`. This will find all changesets that have been created since the last release, and update the version in package.json as specified in those changesets, flattening out multiple bumps to a single package into a single version update.

The `yarn version-packages` command will generate a release commit, which will bump all versions, necessary dependency version changes, and update changelog.mds.

The commands to run are:

```sh
git checkout master && \
git pull && \
git branch -D temp-release-branch && \
git checkout -b temp-release-branch && \
yarn fresh && \
yarn build && \
yarn version-packages && \
yarn format && \
git add . && \
git commit -m "Version packages" && \
git push --set-upstream origin temp-release-branch
```

Once you have run this you will need to make a pull request to merge this back into master.

Finally, make sure you've got the latest of everything locally

```sh
git checkout master && \
git pull && \
yarn
```

#### A quick note on changelogs

The release process will automatically generate and update a `CHANGELOG.md` file, however this does not need to be the only way this file is modified. The changelogs are deliberately static assets so past changelogs can be updated or expanded upon.

In addition, content added above the last released version will automatically be appended to the next release. If your changes do not fit comfortably within the summary of a changelog, we encourage you to add a more detailed account directly into the `CHANGELOG.md`.

#### Backporting Fixes

Occasionally a bug goes undetected for a few versions. When a fix is discovered,
it may need to be applied to all affected versions (depending on the
severity, security considerations, etc). This is called _backporting_.

First, find out the oldest version which was affected. This can be done using
`git blame`, browsing the `CHANGELOG.md`s, etc.

Once we know which version introduced the bug, walk forward through the
`CHANGELOG.md` noting all the _minor_ and _major_ releases made since.

> Example: If a bug was introduced in `14.0.0`, but not discovered until after
> `15.1.1` was released, the list of _minor_ and _major_ releases may look like:
>
> - `14.0.x`
> - `14.1.x`
> - `15.0.x`
> - `15.1.x`

We're going to do a backport and release for the HEAD of every _minor_ and
_major_ release, ignoring any interim patch releases.

> Example: These may be the releases we'd backport to:
>
> - ❌ `14.0.0`
> - ✅ `14.0.1`
> - ✅ `14.1.0`
> - ❌ `15.0.0`
> - ❌ `15.0.1`
> - ✅ `15.0.2`
> - ✅ `15.1.0`

Now, for each release we want to backport to, we follow this process:

1. Checkout the tag of the release

   Let's say the package being patched is `@keystone-alpha/keystone`, then we
   want to run:

   ```sh
   git checkout -b backport-keystone-14.0.1 @keystone-alpha/keystone@14.0.1
   ```

2. Cherry pick the commit across.

   Fix any merge conflicts that might arise.

   NOTE: Make sure the changeset is either regenerated or edited to accurately
   relfect the change to the one package you're bumping, otherwise weirdness
   will happen.

   ```sh
   git cherry-pick abc123123
   ```

3. Bump package versions

   ```sh
   yarn fresh --prefer-offline && \
   yarn build && \
   yarn version-packages && \
   yarn format && \
   git add . && \
   git commit -m "Backport fix"
   ```

4. Do _NOT_ open a PR

   This change is not going to be PR'd into master. Instead we'll later push
   the tag which contains the commits.

   To confirm everything is as expected, look at the git log:

   ```sh
   git log -p
   ```

5. Publish the newly version bumped package

   Note we can't use changesets to do this special publish as it doesn't handle backports.

   ```sh
   (\
   export PACKAGE_NAME=@keystone-alpha/keystone && \
   export OTP_CODE= && \
   cd packages/keystone && \
   yarn publish --tag=backport --otp=$OTP_CODE && \
   export BACKPORTED_VERSION=`npm dist-tag ls $PACKAGE_NAME | grep 'backport' | sed -e 's/backport: //'` && \
   yarn tag remove $PACKAGE_NAME backport --otp=$OTP_CODE && \
   git tag -a "$PACKAGE_NAME@$BACKPORTED_VERSION" -m "$PACKAGE_NAME@$BACKPORTED_VERSION"
   git push --tags \
   )
   ```

     <!-- this was the cd command but we don't have a command to replace the exact bolt part yet    cd `bolt ws $PACKAGE_NAME exec -- pwd | grep pwd | sed -e 's/.*pwd[ ]*//'` && \ w
   -->

   _NOTE: When prompted for "New version", just hit Enter_

6. Confirm it was published

   ```sh
   npm show <PACKAGE_NAME> versions
   ```

### Build Process

Some of the packages in keystone-5 need to compiled before they're published, we use [preconstruct](https://github.com/preconstruct/preconstruct) to do this.

Preconstruct reads from the `packages.json` preconstruct field for configuration, in keystone-5 all we need to do is set the packages that we need to build. Preconstruct reads all of the packages specified in the packages and checks the `main` and `module` fields and generates compiled versions. Preconstruct under the hood uses rollup to bundle all of the modules together which means people can't expose and interact with the modules (e.g. to change internals that could change in a patch version), it's also results in smaller builds because rollup is better at tree shaking than webpack. Preconstruct also uses babel and reads from the babel config in the repo to compile the code, there is one important babel plugin that Preconstruct adds which is `@babel/plugin-transform-runtime` which tells babel to import the helpers that it uses in generated code from a certain place rather than duplicating them in each package.

Preconstruct can generate a couple different types of modules, in keystone-5, we build esm and commonjs modules.

ESM bundles are built for newer bundlers like parcel, rollup and newer versions of webpack which understand ES modules and can build more optimised bundles from them than they can with commonjs.

We also build commonjs builds to run in node (for testing with jest or etc.) and for bundlers which don't understand esm. Preconstruct generates three files for commonjs, a production, development and a file to import those modules. The production one compiles out `process.env.NODE_ENV !== 'production'` checks which are common in front end libraries but `process.env.NODE_ENV` is expensive to check in node if it happens very often.

## Contributors

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore -->
<table>
  <tr>
    <td align="center"><a href="http://www.thinkmill.com.au"><img src="https://avatars3.githubusercontent.com/u/872310?v=4" width="100px;" alt="Jed Watson"/><br /><sub><b>Jed Watson</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=JedWatson" title="Code">💻</a></td>
    <td align="center"><a href="http://jes.st/about"><img src="https://avatars1.githubusercontent.com/u/612020?v=4" width="100px;" alt="Jess Telford"/><br /><sub><b>Jess Telford</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=jesstelford" title="Code">💻</a></td>
    <td align="center"><a href="http://www.timl.id.au"><img src="https://avatars0.githubusercontent.com/u/616382?v=4" width="100px;" alt="Tim Leslie"/><br /><sub><b>Tim Leslie</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=timleslie" title="Code">💻</a></td>
    <td align="center"><a href="https://hamil.town"><img src="https://avatars1.githubusercontent.com/u/11481355?v=4" width="100px;" alt="Mitchell Hamilton"/><br /><sub><b>Mitchell Hamilton</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=mitchellhamilton" title="Code">💻</a></td>
    <td align="center"><a href="https://twitter.com/JossMackison"><img src="https://avatars3.githubusercontent.com/u/2730833?v=4" width="100px;" alt="Joss Mackison"/><br /><sub><b>Joss Mackison</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=jossmac" title="Code">💻</a></td>
    <td align="center"><a href="http://nathansimpson.design"><img src="https://avatars2.githubusercontent.com/u/12689383?v=4" width="100px;" alt="Nathan Simpson"/><br /><sub><b>Nathan Simpson</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=nathansimpsondesign" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/mikehazell"><img src="https://avatars0.githubusercontent.com/u/814227?v=4" width="100px;" alt="Mike"/><br /><sub><b>Mike</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=mikehazell" title="Code">💻</a></td>
  </tr>
  <tr>
    <td align="center"><a href="https://github.com/molomby"><img src="https://avatars0.githubusercontent.com/u/2416367?v=4" width="100px;" alt="John Molomby"/><br /><sub><b>John Molomby</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=molomby" title="Code">💻</a> <a href="https://github.com/keystonejs/keystone-5/issues?q=author%3Amolomby" title="Bug reports">🐛</a></td>
    <td align="center"><a href="https://dominik-wilkowski.com"><img src="https://avatars3.githubusercontent.com/u/1266923?v=4" width="100px;" alt="Dominik Wilkowski"/><br /><sub><b>Dominik Wilkowski</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=dominikwilkowski" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/Noviny"><img src="https://avatars1.githubusercontent.com/u/15622106?v=4" width="100px;" alt="Ben Conolly"/><br /><sub><b>Ben Conolly</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=Noviny" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/jaredcrowe"><img src="https://avatars1.githubusercontent.com/u/4995433?v=4" width="100px;" alt="Jared Crowe"/><br /><sub><b>Jared Crowe</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=jaredcrowe" title="Code">💻</a></td>
    <td align="center"><a href="https://www.linkedin.com/in/gautamsi"><img src="https://avatars2.githubusercontent.com/u/5769869?v=4" width="100px;" alt="Gautam Singh"/><br /><sub><b>Gautam Singh</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=gautamsi" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/lukebatchelor"><img src="https://avatars2.githubusercontent.com/u/18694878?v=4" width="100px;" alt="lukebatchelor"/><br /><sub><b>lukebatchelor</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=lukebatchelor" title="Code">💻</a></td>
    <td align="center"><a href="http://www.ticidesign.com"><img src="https://avatars2.githubusercontent.com/u/289889?v=4" width="100px;" alt="Ticiana de Andrade"/><br /><sub><b>Ticiana de Andrade</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=ticidesign" title="Code">💻</a></td>
  </tr>
  <tr>
    <td align="center"><a href="https://github.com/aghaabbasq"><img src="https://avatars2.githubusercontent.com/u/17919384?v=4" width="100px;" alt="aghaabbasq"/><br /><sub><b>aghaabbasq</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=aghaabbasq" title="Code">💻</a></td>
    <td align="center"><a href="http://ajaymathur.github.io/"><img src="https://avatars1.githubusercontent.com/u/9667784?v=4" width="100px;" alt="Ajay Narain Mathur"/><br /><sub><b>Ajay Narain Mathur</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=ajaymathur" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/mshavliuk"><img src="https://avatars0.githubusercontent.com/u/6589665?v=4" width="100px;" alt="mshavliuk"/><br /><sub><b>mshavliuk</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/issues?q=author%3Amshavliuk" title="Bug reports">🐛</a> <a href="https://github.com/keystonejs/keystone-5/commits?author=mshavliuk" title="Code">💻</a></td>
    <td align="center"><a href="http://www.wesbos.com"><img src="https://avatars2.githubusercontent.com/u/176013?v=4" width="100px;" alt="Wes Bos"/><br /><sub><b>Wes Bos</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=wesbos" title="Documentation">📖</a> <a href="#ideas-wesbos" title="Ideas, Planning, & Feedback">🤔</a></td>
    <td align="center"><a href="https://github.com/vlad-elagin"><img src="https://avatars1.githubusercontent.com/u/28232030?v=4" width="100px;" alt="vlad-elagin"/><br /><sub><b>vlad-elagin</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=vlad-elagin" title="Documentation">📖</a></td>
    <td align="center"><a href="https://github.com/Olya-Yer"><img src="https://avatars3.githubusercontent.com/u/33322677?v=4" width="100px;" alt="Olya-Yer "/><br /><sub><b>Olya-Yer </b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/issues?q=author%3AOlya-Yer" title="Bug reports">🐛</a></td>
    <td align="center"><a href="https://github.com/1337cookie"><img src="https://avatars2.githubusercontent.com/u/15826769?v=4" width="100px;" alt="1337cookie"/><br /><sub><b>1337cookie</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=1337cookie" title="Documentation">📖</a></td>
  </tr>
  <tr>
    <td align="center"><a href="http://madebymike.com.au"><img src="https://avatars0.githubusercontent.com/u/1320567?v=4" width="100px;" alt="Mike"/><br /><sub><b>Mike</b></sub></a><br /><a href="#ideas-MadeByMike" title="Ideas, Planning, & Feedback">🤔</a> <a href="#projectManagement-MadeByMike" title="Project Management">📆</a> <a href="#review-MadeByMike" title="Reviewed Pull Requests">👀</a> <a href="https://github.com/keystonejs/keystone-5/commits?author=MadeByMike" title="Code">💻</a></td>
    <td align="center"><a href="https://jordanoverbye.com"><img src="https://avatars0.githubusercontent.com/u/6265154?v=4" width="100px;" alt="Jordan Overbye"/><br /><sub><b>Jordan Overbye</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=jordanoverbye" title="Code">💻</a> <a href="https://github.com/keystonejs/keystone-5/commits?author=jordanoverbye" title="Documentation">📖</a></td>
    <td align="center"><a href="https://github.com/prvit"><img src="https://avatars1.githubusercontent.com/u/2816799?v=4" width="100px;" alt="prvit"/><br /><sub><b>prvit</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=prvit" title="Documentation">📖</a></td>
    <td align="center"><a href="https://github.com/kennedybaird"><img src="https://avatars1.githubusercontent.com/u/20593811?v=4" width="100px;" alt="Kennedy Baird"/><br /><sub><b>Kennedy Baird</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=kennedybaird" title="Documentation">📖</a></td>
    <td align="center"><a href="http://thiagodebastos.com"><img src="https://avatars0.githubusercontent.com/u/6151341?v=4" width="100px;" alt="Thiago De Bastos"/><br /><sub><b>Thiago De Bastos</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=thiagodebastos" title="Documentation">📖</a></td>
    <td align="center"><a href="https://dcousens.com"><img src="https://avatars0.githubusercontent.com/u/413395?v=4" width="100px;" alt="Daniel Cousens"/><br /><sub><b>Daniel Cousens</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=dcousens" title="Documentation">📖</a></td>
    <td align="center"><a href="http://simonswiss.com"><img src="https://avatars1.githubusercontent.com/u/485747?v=4" width="100px;" alt="Simon Vrachliotis"/><br /><sub><b>Simon Vrachliotis</b></sub></a><br /><a href="#example-simonswiss" title="Examples">💡</a> <a href="#tutorial-simonswiss" title="Tutorials">✅</a> <a href="#video-simonswiss" title="Videos">📹</a></td>
  </tr>
  <tr>
    <td align="center"><a href="https://github.com/Vultraz"><img src="https://avatars0.githubusercontent.com/u/3558659?v=4" width="100px;" alt="Charles Dang"/><br /><sub><b>Charles Dang</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=Vultraz" title="Code">💻</a> <a href="https://github.com/keystonejs/keystone-5/commits?author=Vultraz" title="Documentation">📖</a></td>
    <td align="center"><a href="https://github.com/dzigg"><img src="https://avatars1.githubusercontent.com/u/4436922?v=4" width="100px;" alt="dzigg"/><br /><sub><b>dzigg</b></sub></a><br /><a href="https://github.com/keystonejs/keystone-5/commits?author=dzigg" title="Documentation">📖</a></td>
  </tr>
</table>

<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!

```

```
