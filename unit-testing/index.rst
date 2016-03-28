Unit Testing
============
Unit Testing is normally the forgotten friend of developers. When using MVVM, deciding what and how to unit test certain parts remains tricky with adding in numerous mocked objects to help out and track.

As such Erin was designed to allow easy unit testing where it counts, the IOperation. While the IModelExecute is easy to unit test, even without the IModelExecute the benefit of the IViewModelExecute is shown when unit testing these operations, which return an IResult.