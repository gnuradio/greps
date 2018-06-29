# GREP 0012 -- Replace CppUnit with Boost Unit Testing Framework

[Note: All parts in square brackets are meant to be replaced as part of
authoring a new GREP]

- Original Author: Martin Braun <martin@gnuradio.org>
- Champion: Martin Braun <martin@gnuradio.org>
- Status: Active

History:
- 5-May-2018: Initial Draft
- 21-June-2018: Activated, first conversions merged

## Abstract

As of May 2018, GNU Radio uses the CppUnit library for C++ unit testing (Note:
Python unit tests use something else entirely, and are not relevant to this
GREP).

Boost offers a unit testing framework that is equally capable to do the things
that we require for GNU Radio. Because Boost is already a dependency of GNU
Radio, and much less likely to be removed any time soon, this GREP proposes to
use the Boost Unit Testing Framework (UTF) instead.

## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2018 The GNU Radio Foundation.

## Motivation

Fewer dependencies are always better, unless we take away functionality. In this
case, we can replace one dependency (CppUnit) with one that we already have
(Boost) without losing any functionality.

## Description

There are some key differences between the way CppUnit sets up testing and the
way Boost UTF does:

- In CppUnit, we currently have a single test executable per component (e.g.,
  gr-blocks has one, gr-digital has one, etc.). With Boost UTF, we can keep this
  or very easily move to a more granular test setup, the same way as the Python
  tests work. This lets developers run individual tests more easily (e.g. by
  invoking `ctest -R`).
- With Boost UTF, fewer files are needed. One .cc file per test is sufficient.
  Our current CppUnit setup requires both a .h and a .cc file per test, as well
  as one header and .cc file per top-level component. In general, the macro-driven
  Boost UTF is less verbose than CppUnit for boilerplate code.
- Not surprisingly, CppUnit and Boost use different macros for executing
  assertions.

In order to replace CppUnit with Boost UTF, the following tasks are required:

- Create a CMake macro to more easily create a test from a .cc file. This could
  be rolled into the existing GrTest.cmake file.
- One by one, edit existing `qa_*.cc` files to use Boost UTF instead of CppUnit.
  The changes typically include:
  - Replacing `#includes`
  - Using `BOOST_AUTO_TEST_CASE()` to invoke tests, instead of making tests
    methods of `CppUnit::TestCase`-derived classes
  - Replace the CppUnit assertion macros with the Boost equivalents. This is
    often something that can be done by search and replace, although some tests
    behave differently despite a similar call signature, e.g. `CPPUNIT_ASSERT_DOUBLES_EQUAL`
    looks a lot like `BOOST_CHECK_CLOSE()`, but the latter operates on relative
    errors, and the former on absolute errors.

### Gradual vs. monolithic transition

Since Boost is already a dependency of GNU Radio, there is no cost in gradually
converting the unit tests one by one. This can be a great way for novice
contributors to help with the codebase.

## Appendix A: Required CMake code to enable a Boost UTF test without any additional macros

```cmake
    add_definitions(-DBOOST_TEST_DYN_LINK -DBOOST_TEST_MAIN)
    add_executable(test-gr-rotator ${CMAKE_CURRENT_SOURCE_DIR}/qa_rotator.cc)
    target_link_libraries(
      test-gr-rotator
      gnuradio-runtime
      gnuradio-blocks
      ${Boost_LIBRARIES}
    )
    GR_ADD_TEST(test_blocks_rotator test-gr-rotator)
```


## Appendix B: Reference Patch to change a single unit test

```patch
From d3302146910b3b0f79764aa63be6228aa526aac8 Mon Sep 17 00:00:00 2001
From: Martin Braun <martin.braun@ettus.com>
Date: Sat, 5 May 2018 22:32:37 -0700
Subject: [PATCH] blocks: Replace rotator's QA test framework w/ Boost UTF

---
 cmake/Modules/GrBoost.cmake  |  1 +
 gr-blocks/lib/CMakeLists.txt | 12 +++++++++++-
 gr-blocks/lib/qa_blocks.cc   |  2 --
 gr-blocks/lib/qa_rotator.cc  | 17 ++++++++---------
 gr-blocks/lib/qa_rotator.h   | 41 -----------------------------------------
 5 files changed, 20 insertions(+), 53 deletions(-)
 delete mode 100644 gr-blocks/lib/qa_rotator.h

diff --git a/cmake/Modules/GrBoost.cmake b/cmake/Modules/GrBoost.cmake
index 150009a..e074462 100644
--- a/cmake/Modules/GrBoost.cmake
+++ b/cmake/Modules/GrBoost.cmake
@@ -33,6 +33,7 @@ set(BOOST_REQUIRED_COMPONENTS
     system
     regex
     thread
+    unit_test_framework
 )
 
 if(UNIX AND NOT BOOST_ROOT AND EXISTS "/usr/lib64")
diff --git a/gr-blocks/lib/CMakeLists.txt b/gr-blocks/lib/CMakeLists.txt
index e6eabd8..e626f47 100644
--- a/gr-blocks/lib/CMakeLists.txt
+++ b/gr-blocks/lib/CMakeLists.txt
@@ -315,7 +315,6 @@ if(ENABLE_TESTING)
     ${CMAKE_CURRENT_SOURCE_DIR}/qa_gr_hier_block2_derived.cc
     ${CMAKE_CURRENT_SOURCE_DIR}/qa_blocks.cc
     ${CMAKE_CURRENT_SOURCE_DIR}/qa_block_tags.cc
-    ${CMAKE_CURRENT_SOURCE_DIR}/qa_rotator.cc
     )
 
   add_executable(test-gr-blocks ${test_gr_blocks_sources})
@@ -331,4 +330,15 @@ if(ENABLE_TESTING)
   )
 
   GR_ADD_TEST(test_gr_blocks test-gr-blocks)
+
+  add_definitions(-DBOOST_TEST_DYN_LINK -DBOOST_TEST_MAIN)
+  add_executable(test-gr-rotator ${CMAKE_CURRENT_SOURCE_DIR}/qa_rotator.cc)
+  target_link_libraries(
+    test-gr-rotator
+    gnuradio-runtime
+    gnuradio-blocks
+    ${Boost_LIBRARIES}
+  )
+  GR_ADD_TEST(test_blocks_rotator test-gr-rotator)
+
 endif(ENABLE_TESTING)
diff --git a/gr-blocks/lib/qa_blocks.cc b/gr-blocks/lib/qa_blocks.cc
index c7d76ed..889af39 100644
--- a/gr-blocks/lib/qa_blocks.cc
+++ b/gr-blocks/lib/qa_blocks.cc
@@ -27,7 +27,6 @@
 
 #include <qa_blocks.h>
 #include <qa_block_tags.h>
-#include <qa_rotator.h>
 #include <qa_gr_block.h>
 #include <qa_gr_flowgraph.h>
 #include <qa_set_msg_handler.h>
@@ -38,7 +37,6 @@ qa_blocks::suite()
   CppUnit::TestSuite *s = new CppUnit::TestSuite("gr-blocks");
 
   s->addTest(qa_block_tags::suite());
-  s->addTest(qa_rotator::suite());
   s->addTest(qa_gr_block::suite());
   s->addTest(qa_gr_flowgraph::suite());
   s->addTest(qa_set_msg_handler::suite());
diff --git a/gr-blocks/lib/qa_rotator.cc b/gr-blocks/lib/qa_rotator.cc
index c63d150..ae5a135 100644
--- a/gr-blocks/lib/qa_rotator.cc
+++ b/gr-blocks/lib/qa_rotator.cc
@@ -24,13 +24,12 @@
 #include <config.h>
 #endif
 
+#include <boost/test/unit_test.hpp>
 #include <gnuradio/attributes.h>
-#include <cppunit/TestAssert.h>
-#include <qa_rotator.h>
 #include <gnuradio/blocks/rotator.h>
+#include <gnuradio/expj.h>
 #include <stdio.h>
 #include <cmath>
-#include <gnuradio/expj.h>
 
 // error vector magnitude
 __GR_ATTR_UNUSED static float
@@ -39,8 +38,7 @@ error_vector_mag(gr_complex a, gr_complex b)
   return abs(a-b);
 }
 
-void
-qa_rotator::t1()
+BOOST_AUTO_TEST_CASE(t1)
 {
   static const unsigned	int N = 100000;
 
@@ -66,7 +64,8 @@ qa_rotator::t1()
 	   i, expected.real(), expected.imag(), actual.real(), actual.imag(), evm);
 #endif
 
-    CPPUNIT_ASSERT_COMPLEXES_EQUAL(expected, actual, 0.0001);
+    BOOST_CHECK(std::abs(expected - actual) <= 0.0001);
+    //CPPUNIT_ASSERT_COMPLEXES_EQUAL(expected, actual, 0.0001);
 
     phase += phase_incr;
     if(phase >= 2*M_PI)
@@ -74,8 +73,7 @@ qa_rotator::t1()
   }
 }
 
-void
-qa_rotator::t2()
+BOOST_AUTO_TEST_CASE(t2)
 {
   static const unsigned	int N = 100000;
 
@@ -107,7 +105,8 @@ qa_rotator::t2()
 	   i, expected.real(), expected.imag(), actual.real(), actual.imag(), evm);
 #endif
 
-    CPPUNIT_ASSERT_COMPLEXES_EQUAL(expected, actual, 0.0001);
+    BOOST_CHECK(std::abs(expected - actual) <= 0.0001);
+    //CPPUNIT_ASSERT_COMPLEXES_EQUAL(expected, actual, 0.0001);
 
     phase += phase_incr;
     if(phase >= 2*M_PI)
diff --git a/gr-blocks/lib/qa_rotator.h b/gr-blocks/lib/qa_rotator.h
deleted file mode 100644
index 96fe962..0000000
--- a/gr-blocks/lib/qa_rotator.h
+++ /dev/null
@@ -1,41 +0,0 @@
-/* -*- c++ -*- */
-/*
- * Copyright 2008,2013 Free Software Foundation, Inc.
- *
- * This file is part of GNU Radio
- *
- * GNU Radio is free software; you can redistribute it and/or modify
- * it under the terms of the GNU General Public License as published by
- * the Free Software Foundation; either version 3, or (at your option)
- * any later version.
- *
- * GNU Radio is distributed in the hope that it will be useful,
- * but WITHOUT ANY WARRANTY; without even the implied warranty of
- * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
- * GNU General Public License for more details.
- *
- * You should have received a copy of the GNU General Public License
- * along with GNU Radio; see the file COPYING.  If not, write to
- * the Free Software Foundation, Inc., 51 Franklin Street,
- * Boston, MA 02110-1301, USA.
- */
-
-#ifndef _QA_GR_ROTATOR_H_
-#define _QA_GR_ROTATOR_H_
-
-#include <cppunit/extensions/HelperMacros.h>
-#include <cppunit/TestCase.h>
-
-class qa_rotator : public CppUnit::TestCase
-{
-  CPPUNIT_TEST_SUITE(qa_rotator);
-  CPPUNIT_TEST(t1);
-  CPPUNIT_TEST(t2);
-  CPPUNIT_TEST_SUITE_END();
-
- private:
-  void t1();
-  void t2();
-};
-
-#endif /* _QA_GR_ROTATOR_H_ */
-- 
2.7.4

```
