Getting Started
===============

Install Nuget Package
---------------------

Search for Exrin in Nuget and install into the Native Project and the common PCL.

Initialize Framework
--------------------

In your native project, after the Xamarin Forms Init, add in the following code

.. sourcecode:: csharp

    Exrin.Framework.App.Init();

This sets up your project and you are now free to use whatever parts of Exrin you want. If you want to run your entire project on the Exrin Framework, follow all the details below.

Wire Up
-------

In order for Exrin to avoid dependencies on your platform or other packages it creates what we call a wire up to connect Exrin to your projects dependencies.

Injection
~~~~~~~~~

Dependency injection is a major part of Exrin and we wanted you to choose your favorite framework. This is where we wire up your Dependency Injection framework to Exrin.

A well known DI Framework is AutoFac, which will be used in this example. As per the example below, create a Folder in your project of Wire, then a class called Injection that implements IInjection.

**Important Note:** In the Init() method, you must inject IInjection into itself. We call it Injection Inception and its as confusing as its sounds. We are looking at ways to simplify this.

.. sourcecode:: csharp

	public class Injection: IInjection
	{

		private static ContainerBuilder _builder = null;
		private static IContainer Container { get; set; } = null;

		public void Init()
		{
			_builder = new ContainerBuilder();

			_builder.RegisterInstance<IInjection>(this).SingleInstance();
		}

		public void Complete()
		{
			Container = _builder.Build();
		}

		public void Register<T>() where T : class
		{
			_builder.RegisterType<T>().SingleInstance();
		}

		public void Register<I, T>() where T : class, I	where I : class
		{
			_builder.RegisterType<T>().As<I>().SingleInstance();
		}
        
		public T Get<T>() where T : class
		{
			return Container.Resolve<T>();
		}

		public object Get(Type type)
		{
			return Container.Resolve(type);
		}
	}


Navigation Container
~~~~~~~~~~~~~~~~~~~~

The navigation container wires up the Navigation Page for Exrin. This allows Exrin to not be dependant upon any Xamarin Forms version. You must implement the INavigationPage and an example is below.

.. sourcecode:: csharp

    public class NavigationContainer : INavigationPage
    {

        private readonly NavigationPage _page = null;
        public event EventHandler<IPageNavigationArgs> OnPopped;
        private Queue<object> _argQueue = new Queue<object>();
        private AsyncLock _lock = new AsyncLock();

        public NavigationContainer(NavigationPage page)
        {
            _page = page;
            _page.Popped += _page_Popped;
        }

        private void _page_Popped(object sender, NavigationEventArgs e)
        {
            if (OnPopped != null)
            {
                var poppedPage = e.Page as IPage;
                var currentPage = _page.CurrentPage as IPage;
                var parameter = _argQueue.Count > 0 ? _argQueue.Dequeue() : null;
                OnPopped(this, new PageNavigationArgs() { Parameter = parameter, CurrentPage = currentPage, PoppedPage = poppedPage });
            }
        }

        public void SetNavigationBar(bool isVisible, object page)
        {
            var bindableObject = page as BindableObject;
            if (bindableObject != null)
                NavigationPage.SetHasNavigationBar(bindableObject, isVisible);
        }

        public object Page { get { return _page; } }

        public bool CanGoBack()
        {
            return _page.Navigation.NavigationStack.Count > 1;
        }

        public async Task PopAsync(object parameter)
        {
            using (var releaser = await _lock.LockAsync())
            {
                _argQueue.Enqueue(parameter);
                await _page.PopAsync();
            }
        }

        public async Task PopAsync()
        {
            using (var releaser = await _lock.LockAsync())
            {
                await _page.PopAsync();
            }
        }

        public async Task PushAsync(object page)
        {
            await ThreadHelper.RunOnUIThreadAsync(async () =>
            {
                var xamarinPage = page as Page;

                if (xamarinPage == null)
                    throw new Exception("PushAsync can not push a non Xamarin Page");

                await _page.PushAsync(xamarinPage); // Must be run on the Main Thread
            });
        }
    }

Pages
-----

All pages in this framework must implement IPage. It is recommended that all your pages inherit from a single BasePage as per the example.

.. sourcecode:: csharp

    public partial class BasePage : ContentPage, IPage
    {
        public BasePage()
        {
            InitializeComponent();
        }      
    }

When creating your pages you will find it easier to refer to if you create an enum of them. We do this to separate the actual type or implementation of the page to a key used for navigating to it.

.. sourcecode:: csharp

	namespace Mobile.PageLocator
	{
		public enum Authentication
		{
			Pin = 0
		}

		public enum Main
		{
			Main = 0
		}
	}
	

Models
------

In the MVVM pattern, Models are there to host the business logic, data gathering and state recording. We will look into actually performing an action in the Model later, right now we just need to set it up. We recommend you setup a Base Model as per the example below.

.. sourcecode:: csharp

    public class BaseModel: Exrin.Framework.Model
    {
        public BaseModel(IDisplayService displayService, IErrorHandlingService errorHandlingService)
            :base(displayService, errorHandlingService)
        {           
        }
    }


View Models
-----------

View Models are meant to be nothing more than glue code moving events and data between the View (Page) and Model.

Setting up a base View Model is recommended and it will need to have some objects injected into it as per the example.

.. sourcecode:: csharp
	
    public class BaseViewModel : Exrin.Framework.ViewModel
    {        
        public BaseViewModel(IDisplayService displayService, INavigationService navigationService, 
            IErrorHandlingService errorHandlingService, IStackRunner stackRunner)
             : base(displayService, navigationService, errorHandlingService, stackRunner)
        {  
        }
    }

Stacks
------

Stacks are referring to Navigation Stacks. Rather than having modal navigation stacks, we have the ability to create a stack that houses numerous related pages. The most common example for this is an authentication stack and a main stack. One for login, the other as your main app. Some apps only need these 2, others may require several. Exrin has no restrictions on the amount of stacks you can have.

In the stack you must inherit from BaseStack, then Map the ViewModels, Views and Keys to each other. You must also set the default starting page of the stack.

.. sourcecode:: csharp

    public class AuthenticationStack : BaseStack
    {
        IPageService _pageService = null;

        public AuthenticationStack(INavigationService navigationService, IPageService pageService)
            : base(navigationService)
        {
            _pageService = pageService;
            SetContainer(new NavigationContainer(new NavigationPage()));
            ShowNavigationBar = false;
        }

        protected override void MapPages()
        {
            _navigationService.Map(nameof(PageLocator.Authentication.Pin), typeof(PinPage));
        }

        protected override void MapViewModels()
        {
            _pageService.Map(typeof(PinPage), typeof(PinViewModel));
        }

        protected override string NavigationStartPageKey
        {
            get
            {
                return nameof(PageLocator.Authentication.Pin);
            }
        }
    }

At this point we also need to create an enum of the Stacks we are creating to enable us to switch between them later.

.. sourcecode:: csharp

    public enum Stacks
    {
        Authentication = 0,
        Main = 1
    }


Bootstrapper
------------

The last part is bringing it all together in the bootstrapper. Inherit from Exrin.Framework.Bootstrapper and override the InitStacks and InitModels to register or inject what you have setup.

In the base constructor you will see this is where we send the instatiated Injection object and an Action that assigns a page to the MainPage in Xamarin Forms.

.. sourcecode:: csharp

    public class Bootstrapper : Exrin.Framework.Bootstrapper
    {
        public Bootstrapper() : base(new Injection(), (newPage) => { Application.Current.MainPage = newPage as Page; }) { }

        protected override void InitStacks()
        {          
            RegisterStack<AuthenticationStack>(Stacks.Authentication);
            RegisterStack<MainStack>(Stacks.Main);
        }

        protected override void InitModels()
        {
            _injection.Register<IPinModel, PinModel>();
            _injection.Register<IMainModel, MainModel>();
        }
    }


Launching the App
-----------------

From here we are finally at a point where we will put our line of code in the App.cs file and start the app using Exrin.

.. sourcecode:: csharp

    public App()
    {
        new Bootstrapper().Init().Get<IStackRunner>().Run(Mobile.Stacks.Authentication);
    }
	
IViewModelExecute
-----------------

In order to add functionality to your ViewModel, Exrin requires that you use the IViewModelExecute for any Commands. As in the example below you will see the command for when a key is pressed on the Pin Screen in our sample app. It contains nothing more than glue code to connect to the appropriate IViewModelExecute.

.. sourcecode:: csharp

    private IRelayCommand _keyPressCommand = null;
    public IRelayCommand KeyPressCommand
    {
        get
        {
            return _keyPressCommand ??
                    (Execution.ViewModelExecute(new PinLoginViewModelExecute(Model, Keypad.BackCharacter)));
        }
    }

You need to create the class PinLoginViewModelExecute, which houses the numerous operations and timeout setting for the operations.

.. sourcecode:: csharp

    public class PinLoginViewModelExecute : BaseViewModelExecute 
    {
        public PinLoginViewModelExecute(IPinModel model, string backCharacter)
        {
            TimeoutMilliseconds = 10000;
            Operations.Add(new PinLoginOperation(model, backCharacter));
        }
    }

Next you need to create an IOperation to add to the operations lists. This allows you define the Operation and optional rollback function.

IModelExecute
-------------

Exrin optionally allows you to wrap each model function in an IModelExecute to handle model wide the Timeout and Error handling.

.. sourcecode:: csharp

    public Task<bool> IsPinValid()
	{
		return Execution.ModelExecute(new IsPinValidModelExecute(Pin));
	}

Next you need to create your ModelExecute class that inherits from : IModelExecute<T> with T being the return type of the function. Then define the operation as per the example below.

.. sourcecode:: csharp

    public IOperation<bool> Operation
    {
        get
        {
            return new Operation<bool>()
            {
                Function = () =>
                {
                    if (_pin.Length == 4)
                        return Task.FromResult(true);
                    else
                        return Task.FromResult(false);
                }
            };
        }
    }

Nesting Files
-------------
Due to the need for more classes than usual with this approach it is recommended you nest your files using Visual Studio's DependantUpon tag. Because Visual Studio doesn't have an inbuilt way to manage this, using the extension () is recommended.

Summary
-------
Be sure to look at Unit Testing next to see the benefits of the IViewModelExecute and IModelExecute setup.