# study golang


package main

import (
	"fmt"
	"strings"
)

func main() {
	//defer_call()


	a := []string{
		"a",
		"b",
	}
	join := strings.Join(a, ",")
	fmt.Println(join)

}
