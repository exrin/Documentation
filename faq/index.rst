Frequently Asked Questions
==========================

Inject Platform Specific Dependencies
----------------------------------------
In all of the samples given of Exrin, you will noticed that the IInjectionProxy class is instantiated within the Bootstrapper class, which is in the PCL. 

To injection platform specific dependencies you need to push the instantiation of the IInjectionProxy class up to the platform and pass it as an instance down into the bootstrapper.

.. sourcecode:: csharp

    // In the native project
    injection.Register<InterfaceType, InstanceType>();

    LoadApplication(new App(injection));

    

.. sourcecode:: csharp

    public class App : Application
    {
        public App(IInjection injection)
        {
           new Bootstrapper(injection).Init();
        }

.. sourcecode:: csharp

    public class Bootstrapper : Exrin.Framework.Bootstrapper
    {
        public Bootstrapper(IInjection injection) : base(injection, (newView) => { Application.Current.MainPage = newView as Page; }) { }

