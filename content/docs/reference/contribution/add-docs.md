---
title: Add documentation
weight: 10
---

To contribute new documentation to {{< reuse "docs/snippets/product-name.md" >}}, follow these steps:

- Identify the audience and intended use for the content.
- Choose the type of content you wish to contribute.
- Decide on a name, location, and other front matter for your content
- Write your contribution following the {{< reuse "docs/snippets/product-name.md" >}} contribution guidelines.
- Submit your contribution as a PR to the `k8sgateway/k8sgateway.io` GitHub repository.
- Follow the review process until your contribution is merged.


## Audience and intended use

The best documentation starts by knowing the intended readers, their knowledge, and what you expect them to do with the information. Otherwise, you cannot determine the appropriate scope and depth of information to provide, its ideal structure, or the necessary supporting information. The following examples show this principle in action:

- The reader needs to perform a specific task: Provide the set of steps that are required to accomplish the task. 

- The reader must understand a concept before they can perform a task: Before the task, tell them about the prerequisite information and provide a link to it.

- The reader needs to make a decision: Provide the conceptual information necessary to know when to make the decision, the available options, and when to choose one option instead of the other.

- The reader needs to understand complex feature relationships: Provide a diagram showing the relationships, rather than writing multiple pages of content that is tedious to read and understand.

After you identified your audience and the information that they need, select a suitable [content type](#content-type). 


## Content types
When you understand the audience and the intended use for the content that you want to add, you can choose the content type that best addresses the needs of the audience. The following table shows the supported content types, their intended audiences, and the goals each type strives to achieve:

|Content type |	Goals | Audiences |
| -- | -- | -- |
|Concepts | Concepts explain key principles of {{< reuse "docs/snippets/product-name.md" >}}, its functionality, features, and underlying technologies. For example, you might want to describe the external authentication flow in {{< reuse "docs/snippets/product-name.md" >}} and how the components interact with each other. Concepts do not include sequences of steps. Instead, links might be provided to corresponding guides that explain how to set up and use the feature that you described in your concept. | New useres or users that are unfamiliar with the project and the feature it provides. |
| Guides | Guides provide the steps or procedure to accomplish a certain task. For example, you might want to write a guide for how to set up external authentication in {{< reuse "docs/snippets/product-name.md" >}}. Guides give only minimal information about how the feature works, but can include links to related concepts. | Users that want to try out {{< reuse "docs/snippets/product-name.md" >}}. | 
| Setup pages | Setup pages focus on steps for how to install, configure, and set up {{< reuse "docs/snippets/product-name.md" >}}. | New or existing uses who want to install {{< reuse "docs/snippets/product-name.md" >}}. 
| Reference pages| Reference pages provide detailed information about certain {{< reuse "docs/snippets/product-name.md" >}} components, its CLI, Helm values, and API. Most reference content is automatically generated from the code. | Users with advanced and deep technical knowledge of the project. | 
| FAQs | FAQs provide quick answers to common questions about the {{< reuse "docs/snippets/product-name.md" >}} project. Generally, the FAQs are not very technical in nature. Answers can link to existing guides, concepts, or other content types for readers to learn more. | Readers with specific questions who are looking for brief answers and resources to learn more. |
| Troubleshooting | Troubleshooting topics provide either general debugging guides or error-specific help. For general debugging, share a process that helps users find, analyze, and resolve the cause of an error. For error-specific topics, provide details on what is happening with example messages, why it's happening, and how to fix the issue. | Users who encounter an error and need to debug. |


## File names and title 

All documentation in the {{< reuse "docs/snippets/product-name.md" >}} project is written in markdown and built by using the static site generator Hugo. In Hugo, the name of the file or folder becomes part of the link to your page. Becaues, of that, it is important to carefully choose the name for the file or folder that you want to add. If the file name consists of multiple words, separate them by hyphens. For example, to add a topic about external authentication, your file name might be `ext-auth.md`. All file names must be lowercase. Keep file and folder names as short as possible to ensure easy cross-linking between topics. 

Pick a title for your topic that has the keywords you want search engines to find. If you want to add a folder or "twistie" to the docs that has multipe sub-topics, create the folder with at least one `_index.md` file. This file has the title of your twistie and can be used to provide general overview information about the section that you want to add. 


## Front matter

Each file must start with a front matter that includes the title and weight of the file. In the following example, the topic shows up as `External authentication` in the left-hand navigation. The weight configures the order of the page relative to the other pages in the directory, and therefore determines the order in which files are displayed in the left-hand navigation. 
```
---
title: External authentication
weight: 20
---
```

## Hugo shortcodes

As mentioned earlier, the documentation in the {{< reuse "docs/snippets/product-name.md" >}} project is built by using the static site generator Hugo. Hugo uses reusable templates, commonly referred to as shortcodes, to display, style, and render site elements, such as tables, videos, or cards and to manage the content for these elements more easily. Supported shortcodes can be found in the [{{< reuse "docs/snippets/product-name.md" >}} project repo](https://github.com/k8sgateway/k8sgateway.io/tree/main/layouts/shortcodes) as well as the [Hextra Hugo theme](https://github.com/imfing/hextra/tree/main/exampleSite/content/docs/guide/shortcodes). 

Review common shortcodes that you find throughout the documentation: 

### reuse
You can use the reuse shortcode to reuse content in multiple places while maintaining a single source of truth. A common use case for this shortcode is a reference to the latest version, product names, or short paragraphs. The source of the reused content is stored as a markdown file in the `assets/docs` directory. 

The following example shows the example syntax for using this shortcode: 
```markdown
The guide includes steps to install {{</* reuse "docs/snippets/product-name.md" */>}} in three ways.
```

The content of `product-name.md` is as follows: 
```
K8sGateway
```

### reuse-image
Similar to the reuse shortcode, the reuse-image shortcode is used to display images in the documentation. Images are stored in the `assets/img` folder.

The following example shows the example syntax for using this shortcode: 

```markdown
{{</* reuse-image src="img/translation.svg" */>}}
```

To support different screen sizes, make sure to use images of type `.svg` only. 


### cards

Use the cards shortcode to display a card in the documentation that links to a specific topic within or outside the documentation. You can read more about this shortcode [here](https://imfing.github.io/hextra/docs/guide/shortcodes/cards/). 

Example card: 

{{< cards >}}
{{< card link="/quickstart" title="Get started" >}}
{{< /cards >}}

Shortcode syntax
```markdown
{{</* cards */>}}
{{</* card link="/quickstart" title="Get started" */>}}
{{</* /cards */>}}
```


### callout

A box that displays important information to the user. You can read more about this shortcode [here](https://imfing.github.io/hextra/docs/guide/shortcodes/callout/).

Example callout: 

{{< callout type="info" >}}
This is a sample callout of type info. 
{{< /callout >}}

Shortcode syntax: 
```markdown
{{</* callout type="info" */>}}
This is a sample callout of type info. 
{{</* /callout */>}}
```


## Create a PR in GitHub

The {{< reuse "docs/snippets/product-name.md" >}} documentation follows the standard GitHub collaboration flow for pull requests (PRs). This well-established collaboration model helps open source projects manage the following types of contributions:

- Add new files to the repository.
- Edit existing files.
- Review the added or modified files.
- Manage multiple release or development branches.

The contribution guidelines assume you can complete the following tasks:

- Fork the [{{< reuse "docs/snippets/product-name.md" >}} repository](https://github.com/k8sgateway/k8sgateway.io/).
- Create a branch for your changes.
- Add commits to that branch.
- Open a PR to share your contribution.

## Review

The documentation maintainers of the {{< reuse "docs/snippets/product-name.md" >}} project will review your pull request to ensure the PR follows the documentation contribution guidelines. Once reviewed and approved by the maintainers, your PR is merged into the documentation codebase and your changes will show up on the documentation site. 