# xUnit - Cleaning side effects

![cover-image](./cover-image.jpg)

I have been working with the file system last few days. As a part of this work, I create directories and files in the file system using code. When it came to unit testing my code, I used the [System.IO.Abstractions.TestingHelpers](https://www.nuget.org/packages/System.IO.Abstractions.TestingHelpers/) library to mock the file system in my unit tests. [System.IO.Abstractions.TestingHelpers](https://www.nuget.org/packages/System.IO.Abstractions.TestingHelpers/) package provides a mock file system with all the methods. Because this package uses an in-memory file system, I didn't need to clean up after. Using an in-memory file system meant I could also make my tests run faster.

But for integration testing, I decided to use the file system(actual), which meant creating directories and files during test runs which I needed to clean up after tests. Because I use xUnit, I needed to do the clean up using one of the following ways.

- [Constructor and Dispose](https://xunit.net/docs/shared-context#constructor) (shared setup/cleanup code without sharing object instances)
- [Class Fixtures](https://xunit.net/docs/shared-context#class-fixture) (shared object instance across tests in a single class)
- [Collection Fixtures](https://xunit.net/docs/shared-context#collection-fixture) (shared object instances across multiple test classes)

xUnit has excellent documentation with examples for each, which you could look into if you are interested in implementing any of those mentioned above. I used [Constructor and Dispose](https://xunit.net/docs/shared-context#constructor) method because I needed to clean up after each test. Unfortunately for me, this caused another issue. Because I was creating unique names for the files inside the tests, I could not delete the file created during the test in the dispose method. So, in the dispose method, I delete everything inside the folder where test files get created. Due to this and because xUnit runs tests in parallel, some tests start to fail during the execution.

> Running unit tests in parallel is a new feature in xUnit.net version 2. There are two essential motivations that drove us to not only enable parallelization but also for it to be a feature that's enabled by default

From [xUnit Documentation](https://xunit.net/docs/running-tests-in-parallel.html)

Luckily for me, we can configure the default behaviour by setting `parallelizeTestCollections` property to `false` in the `xunit.runner.json` file. Not running tests in parallel meant my tests are taking longer to run now. But because these are integration tests, I decided to accept the time penalty for keeping the tests simple.

## Resources

- [xUnit: Shared Context between Tests](https://xunit.net/docs/shared-context)
- [System.IO.Abstractions.TestingHelpers](https://www.nuget.org/packages/System.IO.Abstractions.TestingHelpers)
- [xUnit: Running Tests in Parallel](https://xunit.net/docs/running-tests-in-parallel)
- [xUnit: Configuration Files](https://xunit.net/docs/configuration-files)
