## Source
https://go.dev/doc/articles/wiki/

## Packages exclusively needed
```go
"html/template" 
// Allowing us to keep HTML in a separate file 
// which helps us to change the layout without modifying underlying code.
"net/http"
// Web communication.
"regexp"
// regular expression. Help with security check.
```

## Save and Load Page
Usually a web page is of file stored somewhere. Open a web page means open a formatted file on our browser.
Therefore, we need to _read and write file_ for _save and load Page_.

### Data structures
For operating our file, firstly, we need an explicit file structure.
- Define a structure suitable for Page. `[]byte` is recommended for the body of our web page because that is the type expected by the _io libraries_.

### "os"
We could read and write file easily with standard library `"os"`

### Example:
```go
import (
	"os"
)

type Page struct {
	Title string
	Body  []byte
}

func (p *Page) Save() error {
	filename := p.Title + ".txt"
	return os.WriteFile(filename, p.Body, 0600)
}

func LoadPage(title string) (*Page, error) {
	filename := title + ".txt"
	body, err := os.ReadFile(filename)

	if err != nil {
		return nil, err
	}
	return &Page{Title: title, Body: body}, nil
}
```

## The important handlers
Handlers are an essential part of building web applications in Go. They are used for processing incoming requests and generating appropriate responses.

### `http.ResponseWriter` & `*http.Request`
Arguments for constrcuting a handler.
- `http.ResponseWriter` generates HTTP response.
- `*http.Request` accepts HTTP request

### `func HandleFunc(pattern string, handler func(ResponseWriter, *Request))`
Sets up a routing rule in the web server, stating that any incoming HTTP request with a URL matching the pattern should be handled by the specified `handler` function.

### Example
```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

// Generates a HTTP response 'w' based on the HTTP request recieved.
func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:]) // the string will be wrote to the 'w'.
}

func main() {
    http.HandleFunc("/", handler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## Generate consistent output based on dynamic data: `"template"`
In Go, the template package provides functionality for creating and rendering text-based templates. Templates are used to generate dynamic content such as HTML, XML, plain text, or any other formatted output.
### Parsing template
`func ParseFiles(filenames ...string) (*Template, error)` helps with flexibility. It's used before applying the template to data.
When we have a HTML template for example:
```html
<h1>Editing {{.Title}}</h1>

<form action="/save/{{.Title}}" method="POST">
<div><textarea name="body" rows="20" cols="80">{{printf "%s" .Body}}</textarea></div>
<div><input type="submit" value="Save"></div>
</form>
```
It could be parsed using `template.ParseFiles("edit.html")`. The template variables that surrounded by doube curly braces(`{{}}`) can be dynamically filled with data.

### Applying template to data
`func (t *Template) Execute(wr io.Writer, data any) error` works with `template.ParseFiles(...) (...)`

## Web communication
### func ListenAndServe(addr string, handler Handler) error
    ListenAndServe listens on the TCP network address addr and then calls  
    Serve with handler to handle requests on incoming connections. Accepted
    connections are configured to enable TCP keep-alives.

    The handler is typically nil, in which case the DefaultServeMux is used. 

    ListenAndServe always returns a non-nil error.

## Purify our code
### Closures
Is very helpful in code simplification( functionality abstraction).
Given a function which is used multi-times in functions which have same variable and same returns, it is suggested to enclose the latter into an closure.
For example:
```go
func makeHandler(fn func(http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		m := validPath.FindStringSubmatch(r.URL.Path)
		if m == nil {
			http.NotFound(w, r)
			return
		}
		fn(w, r, m[2])
	}
}
```
In this case, the returned func is **the closure** because the fn of variable of `makeHandler()` is defined outside but enclosed in the returned func.

## Safety
### ListenAndServe()
- `log.Fatal(http.ListenAndServe())`. `log.Fatal()` should be called during connecting. In case anything bad happens, exits the program without causing too many troubles.

### Input Validation

### [Relative path & Absolute path](https://stackoverflow.com/a/24028813)
- Relative path

   A relative path is always relative to the root of the document, so if your html is at the same level of the directory, you'd need to start the path directly with your picture's directory name:

   `"pictures/picture.png"`
   But there are other perks with relative paths:

- dot-slash (`./`)

   Dot (`.`) points to the same directory and the slash (`/`) gives access to it:

   So this:

   `"pictures/picture.png"`
   Would be the same as this:

   `"./pictures/picture.png"`
- Double-dot-slash (`../`)

   In this case, a double dot (`..`) points to the upper directory and likewise, the slash (`/`) gives you access to it. So if you wanted to access a picture that is on a directory one level above of the current directory your document is, your URL would look like this:

   `"../picture.png"`
   You can play around with them as much as you want, a little example would be this:

   Let's say you're on directory A, and you want to access directory X.

   ```
   - root
   |- a
      |- A
   |- b
   |- x
      |- X
	```
   Your URL would look either:
      - Absolute path

         `"/x/X/picture.png"`
      Or:

      - Relative path

         `"./../x/X/picture.png"`

## Spruce up the page
### Using a separate CSS file to reuse CSS style
- Request the CSS file from the server:
   1. Get a file server handler by `http.FileServer()`. The handler handles incoming HTTP request for static files including CSS, and sends the corresponding file content back as the response.
   2. Handle the handler by `http.Handle()`.
   3. Example:
   Here is the file structure in the server. We want to get the `css/styles.css` file and use it across HTML files:
   ![file structure in server](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics/202306301615262.png)
   This is how we get and use it:
   ```go
   fs := http.FileServer(http.Dir("./css"))
   // Get the file server handler,
   // `http.Dir("./css")` specifies the files served come from the "./css" directory.
   http.Handle("/css/", http.StripPrefix("/css", fs))
   // Handle the matching HTTP request.
   // For example, URL == "/css/styles.css" matches the "/css/" pattern 
   // which invokes the function. Then it removes the "/css" from the URL 
   // before passing it to the file server handler fs. 
   // Fanally fs will serve the styles.css file located in the `./css` directory.
   ```
   Result:
   The `./css/styles.css` was sucessfully loaded:
   ![result](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics/202306301651500.png)
