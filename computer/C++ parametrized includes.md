parentTopic:: [[Computer]]

A preprocessor trick to change include path based on some predefined preprocessor constant. In this example `-DBRANCH`.

```cpp
#define BRANCH master
#define STRINGIFY_IMPL(a) #a
#define STRINGIFY(a) STRINGIFY_IMPL(a)
#define CONCAT_IMPL(a, b) a##b
#define CONCAT2(a, b) CONCAT_IMPL(a, b)
#define CONCAT5(a, b, c, d, e) CONCAT2(a, CONCAT2(b, CONCAT2(c, CONCAT2(d, e))))
#define BUILD_BRANCH_SPECIFIC_PATH(pathPrefix, filename) STRINGIFY(CONCAT5(pathPrefix, /, BRANCH, /, filename))

#include BUILD_BRANCH_SPECIFIC_PATH(framework/intel_product, intel_products.inl)
```