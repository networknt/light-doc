Documentation source code for [https://doc.networknt.com](https://doc.networknt.com) site which is generated by the [Hugo](https://github.com/gohugoio/hugo). 

[Stack Overflow](https://stackoverflow.com/questions/tagged/light-4j) |
[Google Group](https://groups.google.com/forum/#!forum/light-4j) |
[Gitter Chat](https://gitter.im/networknt/light-4j) |
[Subreddit](https://www.reddit.com/r/lightapi/) |
[Youtube Channel](https://www.youtube.com/channel/UCHCRMWJVXw8iB7zKxF55Byw) |
[Documentation](https://doc.networknt.com/tool/hugo-site/) |
[Contribution Guide](https://doc.networknt.com/contribute/documentation/) |

### Contributing

For each page in the [document site](https://doc.networknt.com/), there is a source markdown file in this repo under the content folder. If you encounter any issue with the documentation, please open an issue or submit a PR. 

### Branches

* The `develop` branch is where the latest updates are merged into. All developers will be working against this branch. 
* The `master` branch is where the site is built from and is matching with the document site exactly.
* The `next` branch is where we push the changes to release the document site from the master branch

### Build and Test

To view the documentation site locally, you need to clone this repository and install Hugo on your computer.

Then to view the docs in your browser, run Hugo and open up the link http://localhost:1313/ on your browser:

```bash
▶ hugo server

Started building sites ...
.
.
Serving pages from memory
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```
