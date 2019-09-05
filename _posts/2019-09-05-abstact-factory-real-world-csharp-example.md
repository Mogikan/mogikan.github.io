---
layout: post
title:  "Abstract factory C# real world example"
date:   2019-09-05 15:37:15 +0700
categories: Design Patterns
---
Initially reading [GOF Patterns](https://en.wikipedia.org/wiki/Design_Patterns) book was a bit hard for me since I could not understand how to apply patterns in real life.
Other resource were also useless for me since they operate with artificial objects which has nothing to do with real life.
Until you can give a bright/vivid example of pattern in real life you can't say you understand it.
If you don't like some particular pattern you are likely do not understand it.
I will try to give real examples of code even though I also have some gaps in some patterns, but I will try to do my best.
Before you start print this paper [Patterns diagram by McDonald](http://www.mcdonaldland.info/files/designpatterns/designpatternscard.pdf) and add notes on it with real world examples.

There are 3 types of Patterns: ***Creational***, ***Structural***, ***Behavioral***.
* Creational means that they produce instances of some classes.
* Structural means that they describe relationships between classes.
* Behavioral means that they describe instance's behavior inside class implementation.
In this article I'll start with Creational patterns, in particular **Abstract Factory.**

Class diagram:

![Abstract factory]({{site.url}}/images/patterns/AbstractFactory.png)

It's hard to grasp difference between some patterns of different categories, but if you focus on the category you'll understand it better.



## Abstract Factory real world example

Key points
* Provides an interface for creating **families** of objects;
* Creates different types of objects in one interface.

Example:
We had a project with plain Windows Forms. 
Then we acquired license to Component Factory Krypton component (which is now open sourced) in order to implement Ribbon UI.
It comes with nice stylish forms and controls. We still had customers who didn't want to upgrade to Ribbon UI.
So, when you have third party component it's a good idea to have an abstract factory which creates specific forms, dialogs and etc.
According to **Steve McConnell** ***Code complete*** book you should isolate potentially changing code to save money and time when suddenly you will have to switch 3rd Party component.
Otherwise it will take forever to migrate to another library.
So if you are asked to migrate your library or app to some other library before you change all the code make a refactoring which will work the same as before but through AbstractFactory.
Then all you will have to do is to implement new class of Factory and etc.
And yes luckily Krypton is not dead since its creator decided to become employee and opensourced this component. 
But he could also had killed his project in case he didn't open source it.
* Abstract factory which creates all the forms, messageboxes and etc:
~~~~
  public abstract class FrameworkFormFactory
  {
    #region Instance
    private static FrameworkFormFactory defaultFactory = new StandardFrameworkFormFactory();
    private static Dictionary<string, FrameworkFormFactory> factoryList = new Dictionary<string, FrameworkFormFactory>();

    static FrameworkFormFactory()
    {
      factoryList.Add(Framework.Utilities.FormsHelper.IMKFormName, defaultFactory);
      factoryList.Add(Framework.Utilities.FormsHelper.OLAPFormName, defaultFactory);
      factoryList.Add(Framework.Utilities.FormsHelper.RSSFormName, defaultFactory);
      factoryList.Add(Framework.Utilities.FormsHelper.RSSRibbonFormName, defaultFactory);
      factoryList.Add(Framework.Utilities.FormsHelper.RSSSCFormName, defaultFactory);
    }

    public static FrameworkFormFactory Factory
    {
      get
      {
        System.Windows.Forms.Form form = FormsHelper.GetMainForm();
        string val = "";
        if (form != null)
          val = form.GetType().FullName;

        FrameworkFormFactory factory;
        if (factoryList.TryGetValue(val, out factory))
          return factory;

        return defaultFactory;
      }
    }

    internal static void RegisterFormFactory(Type mainFormType, FrameworkFormFactory factory)
    {
      string mainFormClassName = mainFormType.FullName;
      if (factoryList.ContainsKey(mainFormClassName))
        factoryList[mainFormClassName] = factory;
      else
        factoryList.Add(mainFormClassName, factory);
    }
    #endregion

    #region UnhandledForm
    public abstract IUnhandledForm CreateUnhandledForm(String detail, String emailLetter, String bugMessage, Boolean isDomainException);
    #endregion

    #region Border Form
    public abstract IBorderForm CreateBorderForm();
    public abstract IBorderForm CreateBorderForm(Border border);
    #endregion

    #region MessageBoxes
    public abstract DialogResult ShowMessage(string text);
    public abstract DialogResult ShowMessage(IWin32Window owner, string text);
    public abstract DialogResult ShowMessage(string text, string caption);
    public abstract DialogResult ShowMessage(IWin32Window owner, string text, string caption);
    public abstract DialogResult ShowMessage(string text, string caption, MessageBoxButtons buttons);
    public abstract DialogResult ShowMessage(IWin32Window owner, string text, string caption, MessageBoxButtons buttons);
    public abstract DialogResult ShowMessage(string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon);
    public abstract DialogResult ShowMessage(IWin32Window owner, string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon);
    public abstract DialogResult ShowMessage(string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton);
    public abstract DialogResult ShowMessage(IWin32Window owner, string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton);
    public abstract DialogResult ShowMessage(string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton, MessageBoxOptions options);
    public abstract DialogResult ShowMessage(IWin32Window owner, string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton, MessageBoxOptions options);
    public abstract DialogResult ShowMessage(string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton, MessageBoxOptions options, bool displayHelpButton);
    public abstract DialogResult ShowMessage(string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton, MessageBoxOptions options, string helpFilePath);
    public abstract DialogResult ShowMessage(IWin32Window owner, string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton, MessageBoxOptions options, string helpFilePath);
    public abstract DialogResult ShowMessage(string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton, MessageBoxOptions options, string helpFilePath, HelpNavigator navigator);
    public abstract DialogResult ShowMessage(string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton, MessageBoxOptions options, string helpFilePath, string keyword);
    public abstract DialogResult ShowMessage(IWin32Window owner, string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton, MessageBoxOptions options, string helpFilePath, HelpNavigator navigator);
    public abstract DialogResult ShowMessage(IWin32Window owner, string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton, MessageBoxOptions options, string helpFilePath, string keyword);
    public abstract DialogResult ShowMessage(string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton, MessageBoxOptions options, string helpFilePath, HelpNavigator navigator, object param);
    public abstract DialogResult ShowMessage(IWin32Window owner, string text, string caption, MessageBoxButtons buttons, MessageBoxIcon icon, MessageBoxDefaultButton defaultButton, MessageBoxOptions options, string helpFilePath, HelpNavigator navigator, object param);    
    #endregion

    #region Message Form
    public abstract IMessageForm CreateMessageForm();
    #endregion

    #region Fill Form
      public abstract IFillEditorForm CreateFillForm(Fill fill);
      public abstract IFillEditorForm CreateFillForm();
    #endregion

    #region Margins Form
    public abstract IMarginsEditorForm CreateMarginsForm();

    public abstract IMarginsEditorForm CreateMarginsForm(Margins margins);
    #endregion

    #region Expression Form
    public abstract IExpressionEditorForm CreateExpressionEditorForm(IExpressionDataService expressionDataService);
    #endregion

    #region Trial Expire Window
    public abstract ITrialExpireWarningForm CreateTrialExpireWarningForm();
    #endregion

    #region Trial Form
    public abstract ITrialForm CreateTrialForm(bool designTime, License license, Type type, string productName, string requirsOption, LicenseHolder licenseHolder, DateTime endTrialPeriod);
    #endregion

    #region Warning Form
    public abstract IWarningForm CreateWarningForm(string productName);
    #endregion

    #region Select Language Form
    public abstract ISelectLanguageForm CreateSelectLanguageForm();
    #endregion

    #region Format Editor Form
    public abstract IFormatEditorForm CreateFormatEditorForm();
    public abstract IFormatEditorForm CreateFormatEditorForm(Text.TextFormat format);
    #endregion

    #region Collection Editor Form
    public abstract ICollectionEditorForm CreateCollectionEditorForm(IServiceProvider provider, ITypeDescriptorContext context);
    #endregion

    #region Controls

    #region Fill Control
      public abstract IFillEditorComponent CreateFillComponent(Fill fill);
    #endregion

    #region Form Control
      public abstract IFormControl CreateFormControl(System.Windows.Forms.Control control);
    #endregion

    #endregion

  }
~~~~
* Factory1 which implements factory with plain Windows Forms
~~~~
  public class StandartFrameworkFormFactory : FrameworkFormFactory
  {
    #region UnhandledForm
    public override IUnhandledForm CreateUnhandledForm(String detail, String emailLetter, String bugMessage, Boolean isDomainException)
    {
      return new StandartUnhandledForm(detail, emailLetter, bugMessage, isDomainException);
    }
    #endregion

    #region BorderForm
    public override Drawing.Design.IBorderForm CreateBorderForm()
    {
      return new BorderForm();
    }

    public override Drawing.Design.IBorderForm CreateBorderForm(Drawing.Border border)
    {
      return new BorderForm(border);
    }
    #endregion

    #region Message Form
    public override IMessageForm CreateMessageForm()
    {
      return new MessageForm();
    }
    #endregion
  
    #region Fill Form
    public override IFillEditorForm CreateFillForm(Fill fill)
    {
      return new FillEditorForm(fill);
    }

    public override IFillEditorForm CreateFillForm()
    {
      return new FillEditorForm();
    }

    #endregion

    #region Margins Form
    public override IMarginsEditorForm CreateMarginsForm()
    {
      return new MarginsEditorForm();
    }

    public override IMarginsEditorForm CreateMarginsForm(Margins margins)
    {
      return new MarginsEditorForm(margins);
    }
    #endregion

    #region Expression Form
    public override IExpressionEditorForm CreateExpressionEditorForm(IExpressionDataService expressionDataService)
    {
      return new ExpressionEditorForm(expressionDataService);
    }
    #endregion

    #region Trial Expire Window
    public override ITrialExpireWarningForm CreateTrialExpireWarningForm()
    {
      return new TrialExpireWarningForm();
    }
    #endregion

    #region Trial Form
    public override ITrialForm CreateTrialForm(bool designTime, License license, Type type, string productName, string requirsOption, LicenseHolder licenseHolder, DateTime endTrialPeriod)
    {
      return new TrialForm(designTime, license, type, productName, requirsOption, licenseHolder, endTrialPeriod);
    }
    #endregion

    #region Warning Form
    public override IWarningForm CreateWarningForm(string productName)
    {
      return new WarningForm(productName);
    }
    #endregion

    #region Select Language Form
    public override ISelectLanguageForm CreateSelectLanguageForm()
    {
      return new SelectLanguageForm();
    }
    #endregion

    #region Format Editor Form
    public override IFormatEditorForm CreateFormatEditorForm()
    {
      return new FormatEditorForm();
    }

    public override IFormatEditorForm CreateFormatEditorForm(Text.TextFormat format)
    {
      return new FormatEditorForm(format);
    }
    #endregion

    #region Collection Editor Form
    public override ICollectionEditorForm CreateCollectionEditorForm(IServiceProvider provider, ITypeDescriptorContext context)
    {
      return new CollectionEditorForm(provider, context);
    }
    #endregion

    #region Controls
    
    #region Fill Control
    public override IFillEditorComponent CreateFillComponent(Fill fill)
    {
      return new FillEditorControl(fill);
    }
    #endregion

    #region Form Control
    public override IFormControl CreateFormControl(System.Windows.Forms.Control control)
    {
      return new FormControl(control);
    }
    #endregion
    
    #endregion

    #region MessageBox
    public override System.Windows.Forms.DialogResult ShowMessage(string text)
    {
      return MessageBox.Show(text);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text)
    {
      return MessageBox.Show(owner,text);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption)
    {
      return MessageBox.Show(text,caption);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption)
    {
      return MessageBox.Show(owner, text, caption);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons)
    {
      return MessageBox.Show(text, caption,buttons);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons)
    {
      return MessageBox.Show(owner,text,caption,buttons);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon)
    {
      return MessageBox.Show(text, caption, buttons,icon);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon)
    {
      return MessageBox.Show(owner,text,caption,buttons,icon);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton)
    {
      return MessageBox.Show(text,caption,buttons,icon,defaultButton);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton)
    {
      return MessageBox.Show(owner,text,caption,buttons,icon,defaultButton);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options)
    {
      return MessageBox.Show(text, caption, buttons, icon, defaultButton,options);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options)
    {
      return MessageBox.Show(owner,text, caption, buttons, icon, defaultButton, options);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, bool displayHelpButton)
    {
      return MessageBox.Show(text,caption,buttons,icon,defaultButton,options,displayHelpButton);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath)
    {
      return MessageBox.Show(text, caption, buttons, icon, defaultButton, options, helpFilePath);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath)
    {
      return MessageBox.Show(owner, text, caption, buttons, icon, defaultButton, options, helpFilePath);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, System.Windows.Forms.HelpNavigator navigator)
    {
      return MessageBox.Show(text, caption, buttons, icon, defaultButton, options, helpFilePath, navigator);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, string keyword)
    {
      return MessageBox.Show(text, caption, buttons, icon, defaultButton, options, helpFilePath, keyword);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, System.Windows.Forms.HelpNavigator navigator)
    {
      return MessageBox.Show(owner, text, caption, buttons, icon, defaultButton, options, helpFilePath, navigator);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, string keyword)
    {
      return MessageBox.Show(owner, text, caption, buttons, icon, defaultButton, options, helpFilePath, keyword);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, System.Windows.Forms.HelpNavigator navigator, object param)
    {
      return MessageBox.Show(text, caption, buttons, icon, defaultButton, options, helpFilePath, navigator, param);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, System.Windows.Forms.HelpNavigator navigator, object param)
    {
      return MessageBox.Show(owner, text, caption, buttons, icon, defaultButton, options, helpFilePath, navigator, param);
    }
    #endregion
  } 
~~~~
Factory2 which implements Factory with Krypton Components type of Forms
~~~~
  public class ModernFrameworkFormFactory : FrameworkFormFactory
  {
    #region UnhandledForm
    public override IUnhandledForm CreateUnhandledForm(String detail, String emailLetter, String bugMessage, Boolean isDomainException)
    {
      return new ModernUnhandledForm(detail, emailLetter, bugMessage, isDomainException);
    }
    #endregion

    #region BorderForm
    public override IBorderForm CreateBorderForm()
    {
      return new ModernBorderForm();
    }

    public override IBorderForm CreateBorderForm(Border border)
    {
      return new ModernBorderForm(border);
    }
    #endregion

    #region Message Form
    public override IMessageForm CreateMessageForm()
    {
      return new ModernMessageForm();
    }
    #endregion

    #region Fill Form
    public override IFillEditorForm CreateFillForm(Fill fill)
    {
      return new Reporting.Windows.Forms.FrameworkForms.ModernFillEditorForm(fill);
    }

    public override IFillEditorForm CreateFillForm()
    {
      return new Reporting.Windows.Forms.FrameworkForms.ModernFillEditorForm();
    }

    #endregion

    #region Margins Form
    public override IMarginsEditorForm CreateMarginsForm()
    {
      return new MarginsEditorForm();
    }

    public override IMarginsEditorForm CreateMarginsForm(Margins margins)
    {
      return new MarginsEditorForm(margins);
    }
    #endregion

    #region Expression Form
    public override IExpressionEditorForm CreateExpressionEditorForm(IExpressionDataService expressionDataService)
    {
      return new ExpressionEditorForm(expressionDataService);
    }
    #endregion

    #region Trial Expire Window
    public override ITrialExpireWarningForm CreateTrialExpireWarningForm()
    {
      return new TrialExpireWarningForm();
    }
    #endregion

    #region Trial Form
    public override ITrialForm CreateTrialForm(bool designTime, License license, Type type, string productName, string requirsOption, LicenseHolder licenseHolder, DateTime endTrialPeriod)
    {
      return new TrialForm(designTime, license, type, productName, requirsOption, licenseHolder, endTrialPeriod);
    }
    #endregion

    #region Warning Form
    public override IWarningForm CreateWarningForm(string productName)
    {
      return new WarningForm(productName);
    }
    #endregion

    #region Select Language Form
    public override ISelectLanguageForm CreateSelectLanguageForm()
    {
      return new ModernSelectLanguageForm();
    }
    #endregion

    #region Format Editor Form
    public override IFormatEditorForm CreateFormatEditorForm()
    {
      return new ModernFormatEditorForm();
    }

    public override IFormatEditorForm CreateFormatEditorForm(TextFormat format)
    {
      return new ModernFormatEditorForm(format);
    }
    #endregion

    #region Collection Editor Form
    public override ICollectionEditorForm CreateCollectionEditorForm(IServiceProvider provider, ITypeDescriptorContext context)
    {
      return new ModernCollectionEditorForm(provider, context);
    }
    #endregion
    
    #region Controls

    #region Fill Control

    public override IFillEditorComponent CreateFillComponent(Fill fill)
    {
      return new Reporting.Windows.Forms.FrameworkForms.ModernFillEditorControl(fill);
    }
    #endregion

    #region Form Control

    public override IFormControl CreateFormControl(System.Windows.Forms.Control control)
    {
      return new ModernFormControl(control);
    }
    #endregion

    #endregion

    #region MessageBox
    public override System.Windows.Forms.DialogResult ShowMessage(string text)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(text);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(owner, text);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(text, caption);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(owner, text, caption);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(text, caption, buttons);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(owner, text, caption, buttons);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(text, caption, buttons, icon);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(owner, text, caption, buttons, icon);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(text, caption, buttons, icon, defaultButton);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(owner, text, caption, buttons, icon, defaultButton);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(text, caption, buttons, icon, defaultButton, options);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(owner, text, caption, buttons, icon, defaultButton, options);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, bool displayHelpButton)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(text, caption, buttons, icon, defaultButton, options, displayHelpButton);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(text, caption, buttons, icon, defaultButton, options, helpFilePath);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(owner, text, caption, buttons, icon, defaultButton, options, helpFilePath);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, System.Windows.Forms.HelpNavigator navigator)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(text, caption, buttons, icon, defaultButton, options, helpFilePath, navigator);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, string keyword)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(text, caption, buttons, icon, defaultButton, options, helpFilePath, keyword);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, System.Windows.Forms.HelpNavigator navigator)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(owner, text, caption, buttons, icon, defaultButton, options, helpFilePath, navigator);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, string keyword)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(owner, text, caption, buttons, icon, defaultButton, options, helpFilePath, keyword);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, System.Windows.Forms.HelpNavigator navigator, object param)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(text, caption, buttons, icon, defaultButton, options, helpFilePath, navigator, param);
    }

    public override System.Windows.Forms.DialogResult ShowMessage(System.Windows.Forms.IWin32Window owner, string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, System.Windows.Forms.MessageBoxDefaultButton defaultButton, System.Windows.Forms.MessageBoxOptions options, string helpFilePath, System.Windows.Forms.HelpNavigator navigator, object param)
    {
      return ComponentFactory.Krypton.Toolkit.KryptonMessageBox.Show(owner, text, caption, buttons, icon, defaultButton, options, helpFilePath, navigator, param);
    }
    
    #endregion

    #region Registering
    internal static void RegisterFactory(Type mainFormType)
    {
      Framework.Windows.Forms.FrameworkFormFactory.RegisterFormFactory(mainFormType, new ModernFrameworkFormFactory());
    }
    #endregion
  }
~~~~