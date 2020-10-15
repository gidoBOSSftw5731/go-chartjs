chartjs
-------

go wrapper for [chartjs](http://chartjs.org)

[![GoDoc](https://godoc.org/github.com/brentp/go-chartjs?status.png)](https://godoc.org/github.com/brentp/go-chartjs)
[![Build Status](https://travis-ci.org/brentp/go-chartjs.svg)](https://travis-ci.org/brentp/go-chartjs)


Chartjs charts are defined purely in JSON, so this library is mostly
structs and struct-tags that dictate how to marshal to JSON. None of the currently
implemented parts are strongly-typed in this library so it can avoid many errors.

The chartjs javascript/JSON api has a [lot of surface area](http://www.chartjs.org/docs/).
Currently, only the options that I use are provided. More can and will be added as I need
them (or via pull-requests).

There is a small amount of code to simplify creating charts.

data to be plotted by chartjs has to meet this interface.
```Go
  type Values interface {
      // X-axis values. If only these are specified then it must be a Bar plot.
      Xs() []float64
      // Optional Y values.
      Ys() []float64
      // Rs are used to size points for chartType `Bubble`. If this returns an
      // empty slice then it's not used.
      Rs() []float64
  }
```

Example
-------

This longish example shows common use of the library.

```Go
package main

import (
	"log"
	"math"
	"os"

	chartjs "github.com/brentp/go-chartjs"
	"github.com/brentp/go-chartjs/types"
)

// satisfy the required interface with this struct and methods.
type xy struct {
	x []float64
	y []float64
	r []float64
}

// Xs is a function to return all X values in the struct of type xy
func (v xy) Xs() []float64 {
	return v.x
}

// Ys is a function to return all Y values in the struct of type xy
func (v xy) Ys() []float64 {
	return v.y
}

// Rs is a function to return all R values in the struct of type xy
func (v xy) Rs() []float64 {
	return v.r
}
//end of required methods

// check is a basic error checking function. 
func check(e error) {
	if e != nil {
		log.Fatal(e)
	}
}

func main() {
	// define two datasets to plot on the same graph
	var xys1, xys2 xy

    // make some example data.
	for i := float64(0); i < 9; i += 0.1 {
		xys1.x = append(xys1.x, i)
		xys2.x = append(xys2.x, i)

		xys1.y = append(xys1.y, math.Sin(i))
		xys2.y = append(xys2.y, 3*math.Cos(2*i))

	}

	// a set of colors to work with.
	colors := []*types.RGBA{
		&types.RGBA{102, 194, 165, 220},
		&types.RGBA{250, 141, 98, 220},
		&types.RGBA{141, 159, 202, 220},
		&types.RGBA{230, 138, 195, 220},
	}

	// a Dataset contains the data and styling info.
	d1 := chartjs.Dataset{
		Data: xys1, 
		BorderColor: colors[1], 
		Label: "sin(x)", 
		Fill: chartjs.False,
		PointRadius: 10,
		PointBorderWidth: 4,
		BackgroundColor: colors[0]}

	d2 := chartjs.Dataset{
		Data: xys2, 
		BorderWidth: 8, 
		BorderColor: colors[3], 
		Label: "3*cos(2*x)",
		Fill: chartjs.False, 
		PointStyle: chartjs.Star}

	// define chart
	chart := chartjs.Chart{Label: "test-chart"}
	
	// Define a chart X axis for the entire chart
	// do this once per x axis
	_, err := chart.AddXAxis(
		chartjs.Axis{
			Type: chartjs.Linear, 
			Position: chartjs.Bottom, 
			ScaleLabel: &chartjs.ScaleLabel{
				FontSize: 22,
				LabelString: "X",
				Display: chartjs.True}})
	check(err)
	
	// Define the dataset's Y axis and their properties
	// do this once per Y axis
	d1.YAxisID, err := chart.AddYAxis(
		chartjs.Axis{
			Type: chartjs.Linear,
			// can be Top, Bottom, Left, or Right
			Position: chartjs.Left,
			ScaleLabel: &chartjs.ScaleLabel{
				LabelString: "sin(x)",
				Display: chartjs.True}})
	check(err)
	chart.AddDataset(d1)

	d2.YAxisID, err := chart.AddYAxis(
		chartjs.Axis{
			Type: chartjs.Linear, 
			Position: chartjs.Right,
			ScaleLabel: &chartjs.ScaleLabel{
				LabelString: "3*cos(2*x)",
				Display: chartjs.True}})
	check(err)
	chart.AddDataset(d2)
	
	// chartjs.False is a pointer to false (the boolean) added
	// "so that we can differentiate between unset and false"
	// -godoc
	chart.Options.Responsive = chartjs.False

	// create and open file to save html to
	// please add error checking for real world use
	wtr, _ := os.Create("example-chartjs-multi.html")
	defer wtr.Close()
	
	// save file to writer
	err = chart.SaveHTML(wtr, nil)
	check(err)
	}
```

The resulting html will have an interactive `<canvas>` element that looks like this.

![plot](https://cloud.githubusercontent.com/assets/1739/20368217/5068a336-ac10-11e6-8d6c-f711c7c71df3.png "example plot")


Live Examples
-------------

[evaluating coverage on high throughput sequencing data](https://brentp.github.io/goleft/indexcov/ex-indexcov-roc.html)

[inferring sex from sequencing coverage on X and Y chroms](https://brentp.github.io/goleft/indexcov/ex-indexcov-sex.html)
