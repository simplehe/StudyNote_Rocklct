## editorconfig

<a>http://editorconfig.org/</a>

在项目开发过程中，有的人喜欢用tab来缩进，有的人喜欢用空格。为了保持缩进风格的一致，可以使用EditorConfig来规范缩进风格，缩进大小，tab长度以及字符集等。

Editorconfig项目由两部分组成，一个是.editorconfig 的文件格式（format）,一个是editorconfig 插件（plugin）

Editorconfig使用.editorconfig文件来设置Python，JavaScript文件的行尾和缩进风格。官方提供的示例代码如下。

<code>
# EditorConfig is awesome: http://EditorConfig.org

# top-most EditorConfig file
root = true

# Unix-style newlines with a newline ending every file
[*]
end_of_line = lf
insert_final_newline = true

# Matches multiple files with brace expansion notation
# Set default charset
[*.{js,py}]
charset = utf-8

# 4 space indentation
[*.py]
indent_style = space
indent_size = 4

# Tab indentation (no size specified)
[Makefile]
indent_style = tab

# Indentation override for all JS under lib directory
[lib/**.js]
indent_style = space
indent_size = 2

# Matches the exact files either package.json or .travis.yml
[{package.json,.travis.yml}]
indent_style = space
indent_size = 2
</code>