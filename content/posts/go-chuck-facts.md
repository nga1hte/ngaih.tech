---
layout: post
title: "Go Chuck Facts"
slug: "gochuckfacts"
date: "2023-01-10"
comments: false
categories:
  - project
tags:
  - rest api
  - golang
  - binary
---

[Go Chuck Facts](https://github.com/nga1hte/go-chuck-facts) is a Command Line Interface (CLI) created using the golang to fetch facts. Facts about the legend Mr. Chuck Norris are fetch from the api endpoint [chucknorris.io](https://api.chucknorris.io). 

The CLI is created using only the standard libraries provided by the go language and is useful for fundamental understanding of how the language operates. The standard package is well documented and provides good information of how code are tied together and abstracted.

## How it Works

Usage of the api endpoint is simple and provides a simple json of facts. The json can be easily parsed using ```json/encoding``` package of go. 

The endpoint can be accessed using:

```
$ curl https://api.chucknorris.io/jokes/random 

```

The response from the endpoint:

```
{
  "categories": [],
  "created_at": "2020-01-05 13:42:29.296379",
  "icon_url": "https://assets.chucknorris.host/img/avatar/chuck-norris.png",
  "id": "Gg-KPU6qTgqlS7Mpk2feFg",
  "updated_at": "2020-01-05 13:42:29.296379",
  "url": "https://api.chucknorris.io/jokes/Gg-KPU6qTgqlS7Mpk2feFg",
  "value": "Camels have a hump because Chuck Norris needed a place to store his kills."
}
```
The CLI basically sends a ```http:GET``` request to the endpoint and parses the ```json``` to extract the value field and then displays it to the standard output.

## The Project Structure

```
go-chuck-facts
    |-- model
    |      |-- fact.go
    |-- client
    |      |-- client.go
    |-- gocf.go

```

```model/fact.go``` contains the struct that will help parse the ```json``` and extract value.

```client/client.go``` contains functions that will fetch and make http requests using the ```net/http``` package

```gocf.go``` is contains ```main()``` and is the entry point to the program. It utilise the ```flag``` package to parse arguments from the ```stdin``` of the terminal.

> This project heavily relies on the blog post [Diving into go by building a cli application](https://dev.to/erybz/diving-into-go-by-building-a-cli-application-28k9).

## model/fact.go:

Since we are only interested in three fields of the ```json``` we create a new datatype ```Fact``` that will contain categories, URL and Value.

We also create a new struct which will contain an array of strings which we will later use to validate the options that are provided in the arguments.
```
//fact.go
...
type Fact struct {
	Categories []string `json:"categories"`
	URL        string   `json:"url"`
	Value      string   `json:"value"`
}

type categories struct {
	Values []string
}
...
```

**The categories can be obtained from this endpoint.**
```
curl https://api.chucknorris.io/jokes/categories
```

**Response:**
```
["animal","career","celebrity","dev","explicit","fashion","food","history","money","movie","music","political","religion","science","sport","travel"]
```

Since categories are unlikely to change, it is a waste to fetch categories from the endpoint everytime. So we populate the categories struct with the values.
We also create some helper methods that we will use to print the categories and also create a map which we will later use to validate the options.
```
//fact.go
...
var Categories = categories{
	Values: []string{"animal", "career", "celebrity", "dev", "explicit", "fashion", "food", "history", "money", "movie", "music", "political", "religion", "science", "sport", "travel"},
}

func (c categories) PrintCategories() string {
	return strings.Join(c.Values, ", ")
}

func (c categories) MapValues() map[string]bool {
	cat := make(map[string]bool)
	for _, val := range c.Values {
		cat[val] = true
	}
	return cat
}
...
```

## client/client.go

We create a const to store the base url as well as use the time package to store a default timeout. We probably don't want the client to keep waiting for slow response so a default timeout is required. We can request a new connection after the timeout.

The ```net/http``` package provides us with client and servers. We first create a ```http.Client``` which will serve as a way to send http request with different parameters and headers. 

```
// client.go
...
const (
	BaseURL              string        = "https://api.chucknorris.io/jokes/random"
	DefaultClientTimeout time.Duration = 30 * time.Second
)

type CNClient struct {
	client  *http.Client
	baseURL string
}

func NewCNClient() *CNClient {
	return &CNClient{
		client: &http.Client{
			Timeout: DefaultClientTimeout,
		},
		baseURL: BaseURL,
	}
}

func (c *CNClient) SetTimeout(d time.Duration) {
	c.client.Timeout = d
}
...
```

The Fetch method utilises the client to create a new get request, we use ```json/encoding``` package to decode the ```json``` response and store in our struct created from ```model/Fact.go``` and return the datatype.
```
// client.go
...
func (c *CNClient) Fetch(category string) (model.Fact, error) {
	url := BaseURL
	if category != "" {
		url = BaseURL + "?category=" + category
	}
	resp, err := c.client.Get(url)
	if err != nil {
		return model.Fact{}, err
	}
	defer resp.Body.Close()
	var factResp model.Fact
	if err := json.NewDecoder(resp.Body).Decode(&factResp); err != nil {
		return model.Fact{}, err
	}
	return factResp, err
}
...
```

## gocf.go
In our ```gocf.go``` file we utilise the ```flag``` package to capture the  flags and then we compare the option givens with our categories map which we create before. We use the ```strings``` package to join our string array into a long string and print it out to the ```stdout```.

```
func main() {
	categories := model.Categories
	cat := categories.MapValues()

	factAmt := flag.Int("n", 1, "Number of facts to fetch")
	category := flag.String("c", "", "categories: "+categories.PrintCategories())
	flag.Parse()

	if _, ok := cat[*category]; !ok {
		*category = ""
	}

	cnClient := client.NewCNClient()
	cnClient.SetTimeout(time.Duration(30) * time.Second)

	facts := []string{}
	for i := 0; i < *factAmt; i++ {
		fact, err := cnClient.Fetch(*category)
		if err != nil {
			log.Println(err)
		}
		facts = append(facts, "> "+fact.Value)
	}

	fmt.Printf("%s\n", strings.Join(facts, "\n"))
}

```

> That't it.



