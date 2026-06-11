isTopic:: true

# Homepage

```dataviewjs
// If a page has and embed field, we will embed it here, in our homepage.
for (let page of dv.pages()) {
    if (!page.embed) {
        continue
    }

    var paragraph = ""
    if (page.embedParagraph != null) {
        paragraph = "#" + page.embedParagraph
    }

    const link_string = `![[${page.file.path}${paragraph}]]`
    dv.paragraph( `[[${page.file.path}]]`)
    dv.paragraph(link_string)
}
```

```dataviewjs
let list = null;
const homeFile = dv.current().file.path

function validateAnnotations(page) {
    // Skip special pages
    if (page.file.name === "Scratch") return null;

    // Check the mere existance of the annotations
    const isTopic = page.isTopic === true;
    const hasTopic = page.parentTopic != null;
    if (!isTopic && !hasTopic) return "must have either 'parentTopic' or 'isTopic' property";

    // At this point we either have a topic or we are a topic
    if (hasTopic) {
        // A topic must be a link to another page
        const hasCorrectTopic = page.parentTopic && typeof page.parentTopic === "object" && "path" in page.parentTopic;
        if (!hasCorrectTopic) return "'parentTopic' property is not a link";

        // The link must be valid
        const topicPage = dv.page(page.parentTopic.path);
        if (!topicPage) return "invalid topic link";

        // The topic page must actually be a topic page
        if (!topicPage.isTopic) return "topic page is not annotated as a topic";

        // By default, the topic page must be in the same directory. This can be turned off by a special field
        // on the topic page.
        if (!topicPage.allowChildrenPagesInDifferentDirs) {
            const topicDir = page.parentTopic.path.split("/").slice(0, -1).join("/");
            const currentDir = page.file.path.split("/").slice(0, -1).join("/");
            if (topicDir !== currentDir && !isTopic) return "topic page is in different directory";
        }
    }

    // All checks passed
    return null;
}

for (let page of dv.pages()) {
    var errorMsg = validateAnnotations(page)
    if (errorMsg != null) {
        if (list == null) {
            dv.header(2, "Annotation problems");
            dv.el("span", "These pages have annotation problems:");
            list = dv.el("ul", "");
        }
        dv.el("li", page.file.link + " (" + errorMsg + ")", list);
    }
}
```
