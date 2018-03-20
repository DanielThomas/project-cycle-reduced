Demonstration of external/project dependency cycles.

To reproduce, run either:

# `./gradlew :c:dependencies`
# `./gradlew :c:printCompile`

And observe that the external dependency is selected, and is used in preference to treating it as a project dependency and failing with a cyclic dependency.