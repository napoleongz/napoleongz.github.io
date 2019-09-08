package main

import (
	"fmt"
	"strings"
)

func main() {

	a := []string{
		"a",
		"b",
	}
	join := strings.Join(a, ",")
	fmt.Println(join)

}
