# cpptf

[![Build Status](https://travis-ci.org/ShotaOchi/cpptf.svg?branch=master)](https://travis-ci.org/ShotaOchi/cpptf)
[![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/ShotaOchi/cpptf?branch=master&svg=true)](https://ci.appveyor.com/project/ShotaOchi/cpptf)
[![CRAN Version](https://www.r-pkg.org/badges/version/cpptf)](https://cran.r-project.org/package=cpptf)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## About

cpptf provides header files of Cpp-Taskflow.

Cpp-Taskflow is a modern header-only C++ parallel task programming library, [https://github.com/cpp-taskflow/cpp-taskflow](https://github.com/cpp-taskflow/cpp-taskflow). 

Cpp-Taskflow helps us to quickly write parallel and heterogeneous programs with high performance scalability and simultaneous high productivity.

## Installation
You can install cpptf from GitHub.

Run the following R code to install cpptf.
```r
# install from GitHub
devtools::install_github("ShotaOchi/cpptf")
```

## Usage

### How to use Cpp-Taskflow in sourceCpp function of Rcpp

1. Write **// [[Rcpp::plugins(cpp14)]]** and **// [[Rcpp::depends(cpptf)]]** in your code.
1. Include Cpp-Taskflow header file.

An example is shown below.
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

### How to use Cpp-Taskflow in your R package

1. Add **cpptf** and **Rcpp** to Imports fields and LinkingTo fields.
1. Write **CXX_STD = CXX14** in src/Makevars. 
1. Include Cpp-Taskflow header file and Rcpp header file.
