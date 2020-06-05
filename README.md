# cpptf

[![Build Status](https://travis-ci.org/ShotaOchi/cpptf.svg?branch=master)](https://travis-ci.org/ShotaOchi/cpptf)
[![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/ShotaOchi/cpptf?branch=master&svg=true)](https://ci.appveyor.com/project/ShotaOchi/cpptf)
[![CRAN Version](https://www.r-pkg.org/badges/version/cpptf)](https://cran.r-project.org/package=cpptf)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## About

cpptf provides Taskflow header files for R users.

Taskflow is a modern header-only C++ parallel task programming library, [https://github.com/taskflow/taskflow](https://github.com/taskflow/taskflow). 

Taskflow helps us to quickly write parallel and heterogeneous programs with high performance scalability and simultaneous high productivity.

See [https://taskflow.github.io/taskflow/](https://taskflow.github.io/taskflow/) for more information about Taskflow.

## Installation
You can install cpptf from GitHub.

Run the following R code to install cpptf.
```r
# install from GitHub
devtools::install_github("ShotaOchi/cpptf")
```

## Usage

### How to use Taskflow in sourceCpp function of Rcpp

1. Write **// [[Rcpp::plugins(cpp14)]]** and **// [[Rcpp::depends(cpptf)]]** in your code.
1. Include Taskflow header file and Rcpp header file.

### How to use Taskflow in your R package

1. Add **cpptf** and **Rcpp** to Imports fields and LinkingTo fields.
1. Write **CXX_STD = CXX14** in src/Makevars. 
1. Include Taskflow header file and Rcpp header file.

## Example
A simple example is shown below.
```
library(Rcpp)

sourceCpp(code = '

// [[Rcpp::plugins(cpp14)]]
// [[Rcpp::depends(cpptf)]]

#include <Rcpp.h>
#include <taskflow/taskflow.hpp>

// [[Rcpp::export]]
void test()
{
  tf::Executor executor;
  tf::Taskflow taskflow;
  auto [A, B, C, D] = taskflow.emplace(
    [] () { Rcpp::Rcout << "TaskA "; },              //  task dependency graph
    [] () { Rcpp::Rcout << "TaskB "; },              // 
    [] () { Rcpp::Rcout << "TaskC "; },              //          +---+          
    [] () { Rcpp::Rcout << "TaskD "; }               //    +---->| B |-----+   
  );                                                 //    |     +---+     |
                                                     //  +---+           +-v-+ 
  A.precede(B);  // A runs before B                  //  | A |           | D | 
  A.precede(C);  // A runs before C                  //  +---+           +-^-+ 
  B.precede(D);  // B runs before D                  //    |     +---+     |    
  C.precede(D);  // C runs before D                  //    +---->| C |-----+    
                                                     //          +---+          
  executor.run(taskflow).wait();
}
')

test()
```
