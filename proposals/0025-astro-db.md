- Start Date: 2022-08-18
- Reference Issues: <!-- related issues, otherwise leave empty -->
- Implementation PR: <!-- leave empty -->

# Summary

Glob all files from a specific directory (e.g. `/content`) and load them into an in-memory javascript document oriented database (e.g. [LokiJS](https://github.com/techfort/LokiJS)).

# Example

Assuming all content in `/content/blog` had been ingested when running Astro, I see us using it like this when creating dynamic page routes.

```js
export async function getStaticPaths() {
    const posts = await Astro.db(`blog/`)
        .where({
        { published: { $ne: false } },
        })
        .sortBy('published_at', 'desc')
        .fetch()

    return posts.map((post) => {
        return {
            params: {
                slug,
            },
            props: {
                post,
            },
        };
    });
}
```

# Motivation

As we're building our new site, I've noticed an extraordinary amount of globs. We also sometimes need to `import.glob.meta` for content in files where we need the filename as well as the content (JSON files). This leads to a fair amount of quite repetitive code through the codebase.

For example, this is just from the blog microsite.

Loading all our blog posts:

```js
const allPosts = await Astro.glob('../content/blog/posts/**/index.md')
```

All our authors and the filename (to use as a slug), which is also repeated for all our categories:

```js
const authorFiles = import.meta.glob("../../content/blog/authors/*.json");

async function getAuthors() {
  let authors = [];
  for (const path in authorFiles) { // for every file
    const author = await authorFiles[path](); // import the file
    const slug = path.split("/").pop().replace(".json", " ").trim(); // get the filename
    authors.push({ ...(author as object), slug }); // create a new object with author details and filename
  }
  return authors;
}

const allAuthors = await getAuthors();
```

All unique post tags:

```js
const allPosts = await Astro.glob('../content/blog/posts/**/index.md')

let postTags = []
for (const post of allPosts) {
  postTags.push(...post.frontmatter.tags) // manually create an array of all post tags (not unique yet)
}
```

Not an exhaustive list. We still load the posts for each category, each tag, each author, the author and category for each post display page, and more.

# Detailed design

tbc

# Drawbacks

Why should we _not_ do this? Please consider:

- Increased memory usage as we load all contents into a javascript document oriented database.
- Slower builds with the increased memory usage is possible.
- Change of direction from earlier design decisions, perhaps?

# Alternatives

- Consuming the markdown content into a serverless endpoint.
- Building Astro/db as a plugin.
- Using an API-based Headless CMS entirely, and not storing markdown files with the Astro sites.

# Adoption strategy

I believe a new top-level feature on the Astro class will limit the impact on users who do not plan to implement this feature. Something like `Astro.db` provides a clear and new interface to be able to query for any content that has been loaded into the database.

Please consider:

- If we implement this proposal, how will existing Astro developers adopt it?
- Is this a breaking change? Can we write a codemod?
- Can we provide a runtime adapter library for the original API it replaces?
- How will this affect other projects in the Astro ecosystem?

# Unresolved questions

- When running the dev environment, how will changed files update their database-loaded selves?
