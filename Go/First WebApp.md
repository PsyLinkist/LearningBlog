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

###

## Safety
- `log.Fatal(http.ListenAndServe())`. `log.Fatal()` should be called during connecting. In case anything bad happens, exits the program without causing too many troubles.