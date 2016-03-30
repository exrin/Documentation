Unit Testing
============
Unit Testing is normally the forgotten friend of developers. When using MVVM, deciding what and how to unit test certain parts remains tricky with adding in numerous mocked objects to help out and track.

As such Erin was designed to allow easy unit testing where it counts, the IOperation. While the IModelExecute is easy to unit test, even without the IModelExecute the benefit of the IViewModelExecute is shown when unit testing these operations, which return an IResult.

We prefer xUnit as the Unit Testing framework as seen in the examples below.

ViewModel Testing
-----------------

Test the operations used in each View Model. The View Model itself is unlikely to need testing as it is glue code and UI Tests will test that functionality correctly works.

.. sourcecode:: csharp

    public IPinModel PinModel
	{
		get
		{
			return new PinModel(CommonService.DisplayService, CommonService.ErrorHandlingService);
		}
	}

	public Func<IResult, Task> GetOperation()
	{
		return new PinLoginOperation(PinModel, Keypad.BackCharacter).Function;
	}
      
	public static TheoryData<string[]> SimplePin { get { return new TheoryData<string[]>() { new string[] { "1", "2", "3", "4" } }; } }
	public static TheoryData<string[]> SimplePinWithBackspace { get { return new TheoryData<string[]>() { new string[] { "1", "2", Keypad.BackCharacter, "4", "5" } }; } }
	public static TheoryData<string[]> StartingBackspacePin { get { return new TheoryData<string[]>() { new string[] { Keypad.BackCharacter, "2", "1", "4", "5" } }; } }

	[Theory]
	[MemberData(nameof(SimplePin))]
	[MemberData(nameof(SimplePinWithBackspace))]
	[MemberData(nameof(StartingBackspacePin))]
	public void OperationTest(string[] characters)
	{
		var function = GetOperation();

		int count = 0;
		foreach (var character in characters)
		{
			IResult result = new Result(character);

			function(result);

			Assert.NotNull(result);

			if (count == BusinessRules.PinLength -1)
				Assert.Equal(result.ResultAction, ResultType.Navigation);
			else
				Assert.Equal(result.ResultAction, ResultType.None);

		}
	}

Model Testing
-------------

An example of testing an operation on a model.

.. sourcecode:: csharp

    public IOperation<bool> GetOperation(string pin)
	{
		return new IsPinValidModelExecute(pin).Operation;
	}

	[Theory]
	[InlineData("1234")]
	[InlineData("123")]
	[InlineData("12345")]
	public async Task PinLengthTest(string pin)
	{
		Assert.Equal(pin?.Length == BusinessRules.PinLength, await GetOperation(pin).Function());
	}

	[Theory]
	[InlineData(".")]
	[InlineData("....")]
	[InlineData("ghyt")]
	[InlineData("66.8")]
	[InlineData(null)]
	public async Task InvalidCharactersTest(string pin)
	{
		Assert.Equal(false, await GetOperation(pin).Function());
	}