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

			public void Register<I, T>() where T : class, I
												 where I : class
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

 - Pages

 - Models

 - ViewModels

 - Stacks

 - Bootstrapper

 - Framework Initialization


 - IViewModelExecute


 - IModelExecute

