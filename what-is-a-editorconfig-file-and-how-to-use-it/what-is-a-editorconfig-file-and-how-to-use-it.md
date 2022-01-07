# What is a `.editorconfig` file, and how to use it?

![cover-image](./cover-image.jpg)

As individuals, we have our preferences regarding how we would like to format our documents. This could become a problem when we try to work in a team. Think how distracting it would be during a PR review if you need to go through multiple changes which are not related to the intent of the PR. Still, the author's editor causes them because the author formatting preferences are different to the others. Another situation is when you try to make the changes in a hurry. It is easy to add extra spacing or start the open brackets in a new line(not the JavaScript convention). It would be nice to use a way to make sure these issues are fixed when we save the file or at least have a command to do this by our editor. It will be super nice if this is editor independent. You might think, why not use a linter such as ESLint to enforce consistency in a JavaScript development environment. That is a good question. The reason is they only help with specific file types.

Say hello to `.editorconfig`.

What is `.editorconfig`?

> EditorConfig helps maintain consistent coding styles for multiple developers working on the same project across various editors and IDEs. ([EditorConfig](https://editorconfig.org/) website)

## Let's create a `.editorconfig` file for a JavaScript development environment

First, create a file named `.editorconfig` and place it in the root folder. The below file is what I use in my development environment.

```csharp
# EditorConfig is awesome: https://EditorConfig.org

# top-most EditorConfig file
root = true

# All Files
[*]
end_of_line = lf
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true

# YAML Files
[*.{yml,yaml}]
indent_size = 2

# Markdown Files
[*.md]
trim_trailing_whitespace = false
```

Now, all you have to do is, use the format command in your editor to format your documents when you finish editing. I am using Visual Studio Code. EditorConfig extension for Visual Studio Code supports applying some of the rules(i.e. `end_of_line` `insert_final_newline` and `trim_trailing_whitespace`) on file save. Visit [editorconfig.org](https://editorconfig.org/) to find if your editor requires a plugin.

When EditorConfig search for the `.editorconfig` file, it will start with the directory of the opened file. Then it will search every parent directory. EditorConfig will stop searching if the root of the file path is found or EditorConfig finds a file with `root = true` rule.

EditorConfig reads the file from top to bottom. Due to this, the most recent rules found take precedence over the previous rules.

If you would like to find all available properties and their values, you could find them at the [EditorConfig Properties](https://github.com/editorconfig/editorconfig/wiki/EditorConfig-Properties) wiki.

**\*Note:** At the time of writing, all EditorConfig properties and values are case-insensitive.\*

## Further reading

If I tickled your curiosity with EditorConfig to help maintain consistency, I recommend reading the articles below for more information regarding how to set up a `.editorconfig` file for the .Net development workspace.

- [A Very Generic .editorconfig File by Muhammad Rehan Saeed](https://rehansaeed.com/generic-editorconfig-file)
- [EditorConfig Reference for C# Developers by Kent Boogaart](https://kent-boogaart.com/blog/editorconfig-reference-for-c-developers)
