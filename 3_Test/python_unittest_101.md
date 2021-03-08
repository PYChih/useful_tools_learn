# python_unittest_101
###### tags: `tools`
[TOC]
# 知識點
- unittest-intro
- unittest-example
    - basic example
        - TestCase
            - assertTrue
            - assertFalse
            - assertEqual
            - assertRaises
        - Runtest
            - unittest.main()
            - TextTestRunner(verbosity=2).run(suite)
            - verbosity=1 will not show each case
# intro:
- supports
    - test automation, sharing of setup, shutdown code for tests
    - aggregation of tests into collections, independence of the tests from the reporting framework
- important concepts:
    - ==test fixtrue==
        - represents the preparation needed to perform one or more tests
        - any associate cleanup actions
        - example:
            - creating temporary
            - proxy databases
            - directories
            - starting a server process
    - ==test case==
        - smallest unit of testing.
        - it checks for a specific response to a particular set of inputs
        - base class:
            - TestCase: which may be used to create new test cases.
    - ==test suit==
        - a collection of test cases, test suites, or both.
        - used to aggregate tests that should be executed together
    - ==test runner==
        - component which orchestrates the execution of tests and provides the outcome to the user.
        - the runner may use a graphical interface, a textual interface, or return a special value to indicate the results of executing the tests.
- test case and test fixture concepts are supported through the `TestCase` and `FunctionTestCase` classes;
    - TestCase should be used when creating new tests
    - FunctionTestCase can be used when integrating existing test code with a unittest-driven framework.
- When building `test fixtures` using TestCase, the setUp and tearDown methods can be overridden to provide initialization and cleanup for the fixtrue.
- With `functiontestcase` existing functions can be passed to the constructor for these purposes
- When the test is run, the fixture initialization is run first, if it succeeds, the cleanup method is run after the test has been executed, regardless of the outcome of the test.
- Each instance of the TestCase will only be used to run a single test method, so a new fixture is created for each test.
- `Test suites` are implemented by the testsuite class.
    - this class allows individual tests and test suites to be aggregated
    - when the suite is executed, all tests added directly to the suite and in `child` test suites are run.
- `test runner` is an object that provides a single method, run(), which accepts a `TestCase` or `TestSuite` Object as a parameter, and returns a result object.
- `The class TestResult` is provided for use as the result object.
    - unittest provides the `texttestRunner` as a example test runner which reports test results on the standard error stream by default
    - `alternate runners` can be implementsed for other environments without any need to derive from a specific class.
# unittest-example
- basic example: test string method
    - [basic_example.py]
- a testcase is created by subclassing `unittest.TestCase`
- three individual tests are defined with methods ==whose names atart with the letters test==.
- crux of each test
    - assertEqual()
        - to check for an expected result
    - assertFalse()
        - to verify a condition
    - assertRaises()
        - to verify that a specific exception gets raised.
    - these methods are used instead of the assert statement so the test runner can accumulate all test results and produce a report
- final block shows a simple way to run the test.
    - unittest.main()
    - output that looks like this:
    - ![](https://i.imgur.com/f2rp5b7.png)
- instead of `unittest.main()`
    ```
    suite = unittest.TestLoader().loadTestsFromTestCase(TestStringMethods)
    unittest.TextTestRunner(verbosity=2).run(suite)
    ```
    - output that looks like this:
    - ![](https://i.imgur.com/7HtJDaV.png)

# reference
- [unittest_framework](https://docs.python.org/2/library/unittest.html)