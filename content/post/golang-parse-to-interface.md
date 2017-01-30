+++
date = "2015-11-13T21:31:40-08:00"
title = "Golang Parsing JSON into Interfaces"

+++

Interfaces are an incredibly powerful feature in Go, and JSON is one of the 
most ubiquitous serialization formats in use today. If you’re reasonably 
new to Go you may not have tried to mash the two together yet, but if you 
have, you may have seen an error that looks like `json: cannot unmarshal string
into Go value of type Foo`. Clearly the Go creators must have given us a way 
to leverage interfaces when unmarshalling JSON!

We’re going to use the following short sample code to illustrate how to achieve this magic:

```
package main

import (
	"encoding/json"
	"fmt"
	"math"
)

type Shape interface {
	Name() string
	Area() float64
	Circumference() float64
}

type Square struct {
	Size int `json:"size"`
}

func (s Square) Name() string {
	return "square"
}

func (s Square) Area() float64 {
	return float64(s.Size * s.Size)
}

func (s Square) Circumference() float64 {
	return float64(4 * s.Size)
}

type Circle struct {
	Radius int `json:"radius"`
}

func (c Circle) Name() string {
	return "circle"
}

func (c Circle) Area() float64 {
	return math.Pi * float64(c.Radius*c.Radius)
}

func (c Circle) Circumference() float64 {
	return math.Pi * float64(c.Radius*2)
}

type Scene struct {
	Shapes map[string]Shape `json:"shapes"`
}

func main() {
	data := []byte(`{"shapes":{
		"square": {
			"size": 2
		},
		"circle": {
			"radius": 1
		}
	}}`)
	s := Scene{}
	err := json.Unmarshal(data, &s)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(s.Shapes)
}
```

Hopefully it’s reasonably clear why it’s impossible to unmarshal into an 
interface. Interfaces have no fields and no concrete underlying data structure
into which to unmarshal the JSON. They are simply a definition of behaviour 
(a collection of methods) required to satisfy a particular use case. So how 
would we go about using the power and flexibility of interfaces during JSON 
unmarshalling?

If you’ve poked around the `encoding/json` package, you may have noticed it has 
an interface of its own that looks interesting. It’s the `Unmarshaller`. This 
interface provides a way for structs to define their own unmarshalling logic. 
Still we run into the problem that interfaces are not concrete, so our 
interface cannot possibly implement `Unmarshaller` itself.

However, if we hoist the concept up one level, what if we define a type for 
our Scene.Shapes map. It might look like:

```
type ShapeMap map[string]Shape

type Scene struct {
	Shapes ShapeMap `json:"shapes"`
}
```

Simple right? Also not quite enough yet. All we’ve done so far is create 
another layer of indirection but we’re still, at the end of the day, 
attempting to unmarshal JSON into an interface. So what about that 
`Unmarshaller` interface? The `encoding/json` library detects if a struct 
implements the interface and will automatically delegate unmarshalling, 
so what would our `UnmarshalJSON` function for `ShapeMap` look like? Let’s 
take a look:

```
type ShapeMap map[string]Shape

func (sm *ShapeMap) UnmarshalJSON(data []byte) error {
	shapes := make(map[string]json.RawMessage)
	err := json.Unmarshal(data, &shapes)
	if err != nil {
		return err
	}
	result := make(ShapeMap)
	for k, v := range shapes {
		switch k {
		case "square":
			s := Square{}
			err := json.Unmarshal(v, &s)
			if err != nil {
				return err
			}
			result[k] = s
		case "circle":
			c := Circle{}
			err := json.Unmarshal(v, &c)
			if err != nil {
				return err
			}
			result[k] = c
		default:
			return errors.New("Unrecognized shape")
		}
	}
	*sm = result
	return nil
}
```

Within our `UnmarshalJSON` we partially unmarshal the map making use of 
`json.RawMessage` to leave the data associated with a shape in serialized form
until we know what type of shape it is by inspection of the map keys. We loop 
over this partially unmarshalled map, and unmarshal each shape into the 
appropriate struct based on the value of the key. Finally, the line 
`*sm = result` is something we don’t see a lot of Go, there’s rarely call 
for it. Here we are dereferencing the pointer to the receiver struct and 
assigning the value of the result map to it. This gives the receiver the value 
of result in its entirety. If the receiver already contained some state, it 
would get replaced.

Providing a full final example, this now allows us to unmarshal JSON into an 
interface type via a little indirection and use of a couple of handy 
`encoding/json` features, the `Unmarshaller` interface, and `RawMessage`.

```
package main

import (
	"encoding/json"
	"errors"
	"fmt"
	"math"
)

type Shape interface {
	Name() string
	Area() float64
	Circumference() float64
}

type Square struct {
	Size int `json:"size"`
}

func (s Square) Name() string {
	return "square"
}

func (s Square) Area() float64 {
	return float64(s.Size * s.Size)
}

func (s Square) Circumference() float64 {
	return float64(4 * s.Size)
}

type Circle struct {
	Radius int `json:"radius"`
}

func (c Circle) Name() string {
	return "circle"
}

func (c Circle) Area() float64 {
	return math.Pi * float64(c.Radius*c.Radius)
}

func (c Circle) Circumference() float64 {
	return math.Pi * float64(c.Radius*2)
}

type ShapeMap map[string]Shape

func (sm *ShapeMap) UnmarshalJSON(data []byte) error {
	shapes := make(map[string]json.RawMessage)
	err := json.Unmarshal(data, &shapes)
	if err != nil {
		return err
	}
	result := make(ShapeMap)
	for k, v := range shapes {
		switch k {
		case "square":
			s := Square{}
			err := json.Unmarshal(v, &s)
			if err != nil {
				return err
			}
			result[k] = s
		case "circle":
			c := Circle{}
			err := json.Unmarshal(v, &c)
			if err != nil {
				return err
			}
			result[k] = c
		default:
			return errors.New("Unrecognized shape")
		}
	}
	*sm = result
	return nil
}

type Scene struct {
	Shapes ShapeMap `json:"shapes"`
}

func main() {
	data := []byte(`{"shapes":{
		"square": {
			"size": 2
		},
		"circle": {
			"radius": 1
		}
	}}`)
	s := Scene{}
	err := json.Unmarshal(data, &s)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(s.Shapes)
}
```
