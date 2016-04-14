Frequently Asked Questions
==========================

Injection Platform Specific Dependencies
----------------------------------------
In all of the samples given of Exrin, you will noticed that the IInjection class is instantiated within the Bootstrapper class, which is in the PCL. 

To injection platform specific dependencies you need to push the instantiation of the IInjection class up to the platform and pass it as an instance down into the bootstrapper.

.. sourcecode:: csharp

    public sealed partial class MainPage
    {
        public MainPage()
        {
            this.InitializeComponent();

			IInjection injection = new Injection();

	        injection.Register<InterfaceType, InstanceType>();

            LoadApplication(new App(injection));
        }
    }
    

.. sourcecode:: csharp

    public class App : Application
    {

        public App(IInjection injection)
        {
           new Bootstrapper(injection).Init().Get<IStackRunner>().Run(Stacks.Authentication);
        }

.. sourcecode:: csharp

    public class Bootstrapper : Exrin.Framework.Bootstrapper
    {
        public Bootstrapper(IInjection injection) : base(injection, (newView) => { Application.Current.MainPage = newView as Page; }) { }

