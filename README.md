
Deep Exponential Family
==========================

Reference
---------

[Deep Exponential Families](https://github.com/Blei-Lab/Publications/blob/master/2015_RanganathTangCharlinBlei/2015_RanganathTangCharlinBlei.pdf)
by Rajesh Ranganath, Linpeng Tang, Laurent Charlin, and David M. Blei, AISTATS 2015.


Requirements
------------

* armadillo
* boost
* OpenMP
* GSL
* g++ >= 4.7

Setup on Luc's Laptop
---------------------

These are Luc's specific instructions. Required if you have apple/xcode-installed gcc in addition to brew-installed gcc. Use at your own risk.

Installing gcc and g++:
`brew install gcc --without-multilib`

Installing C++ libs (minus boost):
`brew install gsl pkg-config armadillo`

Installing boost:
`brew install boost --build-from-source --cc=gcc-7`


Boost Problems and Solutions
----------------------------

Everyone has a different OS setup. Mine happens to have boost problems.

There are a few compile-time errors which require some boost tweaking.

Error:
```
Undefined symbols for architecture x86_64:
  "boost::program_options::to_internal(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)", referenced from:
  ...
```
We're already included the boost_program_options lib in our wscript, so
this is likely because boost was compiled by clang instead of gcc,
unless it's because it was compiled without c++11 support. In either case,
the fix for me was uninstalling with `brew uninstall boost` and reinstalling.

Solution:
`brew install boost --build-from-source --cc=gcc-7`

See:
https://github.com/Homebrew/legacy-homebrew/issues/30512#issuecomment-47991795

Problem:
```
In file included from ../utils.hpp:16:0,
                 from ../def_main.cpp:1:
/usr/local/include/boost/property_tree/ptree_serialization.hpp: In function 'void boost::property_tree::detail::load_children(Archive&, boost::property_tree::basic_ptree<K, D, C>&)':
/usr/local/include/boost/property_tree/ptree_serialization.hpp:66:24: error: 'library_version_type' in namespace 'bsa' does not name a type
             const bsa::library_version_type library_version(
                        ^~~~~~~~~~~~~~~~~~~~
```

Add this to `../utils.hpp` line 16 (above the include for ptree_serialization):
```
#include <boost/serialization/type_info_implementation.hpp>
#include <boost/archive/basic_archive.hpp>
```

See:
http://www.freeorion.org/forum/viewtopic.php?p=76993#p76993

Problem:
```
Undefined symbols for architecture x86_64:
  "boost::log::v2s_mt_posix:: <snipped many long lines about boost log symbols>
  ...
```

Fix, added to `wscript`:
```
conf.check(compiler='cxx',lib='boost_log-mt', uselib_store='LOG_MT')
```

See:
https://stackoverflow.com/questions/28973565/boost-logging-fails-to-compile-on-osx
https://stackoverflow.com/questions/36117306/link-errors-using-homebrews-boostlog-in-osx-el-capitan

Instructions to Build and Run
-----------------------------

Configuring: 
`./waf configure`

Building: 
`./waf build`
(binary is `build/def_main`)

Running: `def` reads its options from a config file and from the command
line. `./build/def_main --help` shows the command line options. 

We give a full example below (including a sample config file) of a def
running a dataset of wikipedia articles.

Input Format for Text
---------------------

The header:
```
n_examples n_words
```

Followed by `n_examples` examples, each has two lines:
```
example_ind example_words
word_ind0 word_count0 word_ind1 word_count1 ...
```


Comprehensive Example
---------------------

0. Data. We have pre-processed a corpus of wikipedia articles containing
1000 train articles, 500 validation and, 500 test articles. The data are
available in folder [wikpedia](wikipedia/)

0. Configuration file. The configuration file will be read by `def` and contains all
options regarding the model to be trained (e.g., number of layers, size of
layers, distribution of global and local variables).  Example
configuration for the wikipedia dataset is available in
folder [wikipedia](wikipedia/def_wikipedia_50_25_10.ini). Note that this file will look for the dataset in a directory specified by the WIKIPEDIA_DEF environment variable.

0. Running. Here is an example invocation:

`./build/def_main --v=3 --folder=experiments/def_wikipedia --algo=rmsprop --rho=.2 --samples=64 --max_examples=1000000 --model=wikipedia/def_wikipedia_50_25_10.ini --batch=10000 --batch_order=rand --threads=5 --test_interval=5 --iter=2000`

The above settings (including the values provided in the configuration file)
are the ones we used in the paper. We have found these settings to be useful
across several datasets.


Topic visualization
-------------------

We show some of the tools that we have used to explore the def fits in this
[DEF IPython Notebook](http://nbviewer.ipython.org/github/Blei-Lab/deep-exponential-families/blob/master/wikipedia/def_wikipedia_visualization.ipynb).

