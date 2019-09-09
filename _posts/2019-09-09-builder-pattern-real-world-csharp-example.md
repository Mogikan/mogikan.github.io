---
layout: post
title:  "Builder design pattern C# real world example"
date:   2019-09-09 15:37:15 +0700
categories: Design Patterns
---
In the previous post I provided [real world example of Abstract Factory pattern]({% post_url 2019-09-05-abstact-factory-real-world-csharp-example %}).
In today article I will provide real world example of another *creational* pattern in particular Builder pattern.
This pattern idea primarily comes from the following logic. 
When you have several changing parts (which can also exist or not) you will have to combine different parts factorial times to get all combination of objects.
In this case OOP would be nonsense since you will have to create lots of classes to cover all the combinations.
With builder pattern you can create all the combinations using one class, which allows you to build parts.
Key points of this pattern:
* Allows to create different objects by calling or not calling **CreateSomePart** method;
* Does not use inheritance to create different objects composed by different parts.

Class diagram:

![Builder pattern]({{site.url}}/images/patterns/Builder.png)

Example:
The following example will be connected with Xamarin.Android. 
On Android there is AlertDialog.Builder class, but it had several major drawbacks. 
It needs callbacks to operate and it collapsed when you hit out of the dialog.
This class itself is a Builder class, but we added a few more ideas and called it AwaitableDialog
~~~~
internal class AwaitableDialog: StyleableAlertDialog
    {
        public AwaitableDialog(Context context) : base(context)
        {            
        }
        public AwaitableDialog SetValidator(Func<bool> validator)
        {
            _okListener.SetValidator(validator);
            _cancelListener.SetValidator(validator);
            return this;
        }
        public new AwaitableDialog SetMessage(string messageRes)
        {
            base.SetMessage(messageRes);
            return this;
        }

        public new AwaitableDialog SetContentView(View view)
        {
            base.SetContentView(view);            
            return this;
        }

        public new AwaitableDialog SetTitle(string title)
        {
            base.SetTitle(title);
            return this;
        }
        public AwaitableDialog SetPositiveButtonText(string positiveButtonText)
        {
            base.SetPositiveButton(positiveButtonText, (s, e) => { });
            var acceptButton = this.GetButton((int)Android.Content.DialogButtonType.Positive);
            _okListener = new NonCollapsableTrivialListener(Context, Android.Content.DialogButtonType.Positive, this);
            acceptButton.SetOnClickListener(_okListener);
            return this;
        }

        public void CallOK()
        {
            this.GetButton((int)Android.Content.DialogButtonType.Positive).CallOnClick();
        }

        public AwaitableDialog SetNegativeButtonText(string negativeButtonText)
        {            
            base.SetNegativeButton(negativeButtonText, (s, e) => { });
            var cancelButton = this.GetButton((int)Android.Content.DialogButtonType.Negative);            
            _cancelListener = new NonCollapsableTrivialListener(Context, Android.Content.DialogButtonType.Negative, this);
            cancelButton.SetOnClickListener(_cancelListener);

            this.DismissEvent += (o, e) =>
            {
                InputMethodManager inputMethodManager = (InputMethodManager)Context.GetSystemService(Android.App.Activity.InputMethodService);
                inputMethodManager.HideSoftInputFromWindow(cancelButton.WindowToken, 0);
            };
            return this;
        }

        public new AwaitableDialog SetIcon(Drawable drawable)
        {
            base.SetIcon(drawable);
            return this;
        }

        internal object SetPositiveButtonText(object p)
        {
            throw new NotImplementedException();
        }

        public new AwaitableDialog Create()
        {
            return this;
        }
        private NonCollapsableTrivialListener _okListener;
        private NonCollapsableTrivialListener _cancelListener;
        
        public Task<ResultObject<AwaitableDialog>> ShowAsync()
        {
            var result = new TaskCompletionSource<ResultObject<AwaitableDialog>>();
            _okListener.PassCompletionSource(result);
            _cancelListener.PassCompletionSource(result);
            Show();
            return result.Task;
        }
        
        private class NonCollapsableTrivialListener : Java.Lang.Object, View.IOnClickListener, IDialogInterfaceOnDismissListener         
        {
            public NonCollapsableTrivialListener(Context context, Android.Content.DialogButtonType dialogButtonType, AwaitableDialog dialog)
            {
                this._dialog = dialog;
                this._context = context;
                this._dialogButtonType = dialogButtonType;
            }
            public void SetValidator(Func<bool> validator)
            {
                _validator = validator;
            }
            private Func<bool> _validator = () => { return true; };
            private AwaitableDialog _dialog;
            private TaskCompletionSource<ResultObject<AwaitableDialog>> _completionSource;
            private Context _context;
            private DialogButtonType _dialogButtonType;
            public void PassCompletionSource(TaskCompletionSource<ResultObject<AwaitableDialog>> completionSource)
            {
                _completionSource = completionSource;
            }

            public void OnClick(View v)
            {
                switch (_dialogButtonType)
                {
                    case DialogButtonType.Negative:
                        _completionSource.SetResult(new ResultObject<AwaitableDialog>()
                        {
                            ResultValue = _dialog,
                            Status = ResultStatus.Cancelled
                        });
                        _dialog.DismissSafely();
                        break;
                    case DialogButtonType.Neutral:
                        _completionSource.SetResult(new ResultObject<AwaitableDialog>()
                        {
                            ResultValue = _dialog,
                            Status = ResultStatus.Neutral
                        });
                        break;
                    case DialogButtonType.Positive:
                        if (_validator())
                        {
                            _completionSource.SetResult(new ResultObject<AwaitableDialog>()
                            {
                                ResultValue = _dialog,
                                Status = ResultStatus.OK
                            });
                            _dialog.DismissSafely();
                        }
                        break;
                    default:
                        break;
                }

            }
            public void OnDismiss(IDialogInterface dialog)
            {
                Show();
            }

            public void Show()
            {
                _dialog.Show();
            }
        }

~~~~

How to use it

~~~~
            var oldName = item.Name;
            var view = new EnterItemNameView(this);
            view.NameFieldCaption = LocalizationHelper.GetUITableViewActivityNewChartName();//LocalizationHelper.GetUITableViewActivityNewTableName();
            view.NewName = oldName;
            var dialog = new AwaitableDialog(this)
                .SetTitle(LocalizationHelper.GetUITableViewActivityRenameChart())
                .SetPositiveButtonText(LocalizationHelper.GetGeneralOK())
                .SetNegativeButtonText(LocalizationHelper.GetGeneralCancel())
                .SetContentView(view)
                .SetValidator(() =>
                {
                    if (descriptor.Name == view.NewName)
                    {
                        return true;
                    }
                    if (!_dbManager.CurrentDatabase.Template.Charts.Where(x => x != item && x.Name.ToLower() == view.NewName.ToLower()).Any())
                    {
                        return true;
                    }
                    else
                    {
                        view.ShowNameError(LocalizationHelper.GetUIRegistersReportNameIsAlreadyUsed());
                        return false;
                    }
                })
                .Create();
            view.DoneButtonClicked += (s, e) => { dialog.CallOK(); };
            var dialogResult = await dialog.ShowAsync();
            if (dialogResult.Status == Core.ResultStatus.OK)
            {
~~~~
This is almost full version of alert dialog example, by omiting parts **SetSomeProperty** you can create different objects.
This example also demonstrates how chaining make it easy to work with builder, since every **Set...** returns **this** and you can call another **Set** by using dot notation.

For the sake of completeness I will add StyleableAlertDialog code which Awaitable dialog inherits from:

~~~~
    internal class StyleableAlertDialog: Dialog
    {
        #region Fields
        private AlertDialogView _dialogView;
        private Context _context;
        #endregion

        #region Properties/Indexers

        public bool AutoCloseOnPositiveHandler { get; set; } = true;

        #endregion

        #region Methods/Events
        public virtual Button GetButton(int whichButton)
        {
            return _dialogView.GetButton(whichButton);
        }

        #endregion

        #region Constructors/Finalizers

        public StyleableAlertDialog(Context context): base(context)
        {
            this._context = context;
            this._dialogView = new AlertDialogView(context);
            this.RequestWindowFeature((int)WindowFeatures.NoTitle);
            this.SetCanceledOnTouchOutside(false);
            if (VisualSettings.IsSimplifiedMode)
            {
                this.Window.SetLayout(LayoutParams.MatchParent, LayoutParams.MatchParent);
            }
        }

        public AlertDialogView View 
        {
            get 
            {
                return _dialogView;
            }
        }

        #endregion

            /// <summary>
            /// Set message text from resource
            /// </summary>
            /// <param name="messageRes">Resource ID</param>
            /// <returns></returns>
            public StyleableAlertDialog SetMessage(int messageRes)
            {
                return this.SetMessage((String)_context.GetText(messageRes));
            }

            /// <summary>
            /// Set message string
            /// </summary>
            /// <param name="message">Message string</param>
            /// <returns></returns>
            public StyleableAlertDialog SetMessage(String message)
            {
                this._dialogView.SetMessage(message);
                return this;
            }

            /// <summary>
            /// Set title from resource
            /// </summary>
            /// <param name="title">Resource ID</param>
            /// <returns></returns>
            public new StyleableAlertDialog SetTitle(int title)
            {
                return this.SetTitle((String)_context.GetText(title));
            }

            /// <summary>
            /// Set title string
            /// </summary>
            /// <param name="title">Title string</param>
            /// <returns></returns>
            public new StyleableAlertDialog SetTitle(String title)
            {
                this._dialogView.SetTitle(title);
                return this;
            }

            /// <summary>
            /// Set positive button caption from resource and click handler
            /// </summary>
            /// <param name="positiveButtonText">Resource ID for button caption</param>
            /// <param name="handler">Click handler</param>
            /// <returns></returns>
            public StyleableAlertDialog SetPositiveButton(int positiveButtonText, EventHandler<DialogClickEventArgs> handler)
            {
                return this.SetPositiveButton((String)_context.GetText(positiveButtonText), handler);
            }

            /// <summary>
            /// Set positive button caption and click handler
            /// </summary>
            /// <param name="positiveButtonText">Positive button caption</param>
            /// <param name="handler">Click handler</param>
            /// <returns></returns>
            public StyleableAlertDialog SetPositiveButton(String positiveButtonText, EventHandler<DialogClickEventArgs> handler)
            {
                EventHandler dialogHandler = new EventHandler(
                    (o, e) =>
                    {
                        handler.Invoke(o, new DialogClickEventArgs((int)Android.Content.DialogButtonType.Positive));
                        if(AutoCloseOnPositiveHandler)
                        {
                            this.DismissSafely();
                        }
                    }
                    ); 
                this._dialogView.SetPositiveButton(positiveButtonText, dialogHandler);
                return this;
            }

            /// <summary>
            /// Set neutral button caption from resource and click handler
            /// </summary>
            /// <param name="neutralButtonText">Resource ID for button caption</param>
            /// <param name="handler">Click handler</param>
            /// <returns></returns>
            public StyleableAlertDialog SetNeutralButton(int neutralButtonText, EventHandler<DialogClickEventArgs> handler)
            {
                return this.SetNeutralButton((String)_context.GetText(neutralButtonText), handler);
            }

            /// <summary>
            /// Set neutral button caption and click handler
            /// </summary>
            /// <param name="neutralButtonText">Neutral button caption</param>
            /// <param name="handler">Click handler</param>
            /// <returns></returns>
            public StyleableAlertDialog SetNeutralButton(String neutralButtonText, EventHandler<DialogClickEventArgs> handler)
            {
                EventHandler dialogHandler = new EventHandler(
                    (o, e) =>
                    {
                        handler.Invoke(o, new DialogClickEventArgs((int)Android.Content.DialogButtonType.Neutral));
                        this.DismissSafely();
                    }
                    );
                this._dialogView.SetNeutralButton(neutralButtonText, dialogHandler);
                return this;
            }

            /// <summary>
            /// Set neutral button caption and click handler
            /// </summary>
            /// <param name="neutralButtonText">Neutral button caption</param>
            /// <param name="handler">Click handler</param>
            /// <returns></returns>
            public StyleableAlertDialog SetNeutralUnDismissableButton(String neutralButtonText, EventHandler<DialogClickEventArgs> handler)
            {
                EventHandler dialogHandler = new EventHandler(
                    (o, e) =>
                    {
                        handler.Invoke(o, new DialogClickEventArgs((int)Android.Content.DialogButtonType.Neutral));
                    }
                    );
                this._dialogView.SetNeutralButton(neutralButtonText, dialogHandler);
                return this;
            }

            /// <summary>
            /// Set negative button caption from resource and click handler
            /// </summary>
            /// <param name="negativeButtonText">Resource ID for button caption</param>
            /// <param name="handler">Click handler</param>
            /// <returns></returns>
            public StyleableAlertDialog SetNegativeButton(int negativeButtonText, EventHandler<DialogClickEventArgs> handler)
            {
                return this.SetNegativeButton((String)_context.GetText(negativeButtonText), handler);
            }

            /// <summary>
            /// Set negative button caption and click handler
            /// </summary>
            /// <param name="negativeButtonText">Negative button caption</param>
            /// <param name="handler">Click handler</param>
            /// <returns></returns>
            public StyleableAlertDialog SetNegativeButton(String negativeButtonText, EventHandler<DialogClickEventArgs> handler)
            {
                EventHandler dialogHandler = new EventHandler(
                    (o, e) =>
                    {
                        handler.Invoke(o, new DialogClickEventArgs((int)Android.Content.DialogButtonType.Negative));
                        this.DismissSafely();
                    }
                    );
                this._dialogView.SetNegativeButton(negativeButtonText, dialogHandler);
                return this;
            }

            public StyleableAlertDialog SetCancelHandler(EventHandler handler)
            {
                this.CancelEvent += (o, e) => handler(this, EventArgs.Empty);
                return this;
            }

            public StyleableAlertDialog SetIcon(Drawable drawable)
            {
                this._dialogView.SetIcon(drawable);
                return this;
            }

            public new StyleableAlertDialog Create()
            {
                return this;
            }

            public override void Show()
            {
                _dialogView.Restyle();
                base.Show();
                this.SetContentView(_dialogView, new ViewGroup.LayoutParams(this._dialogView.LayoutParameters));
            }

            public StyleableAlertDialog SetItems(string[] items, EventHandler<DialogClickEventArgs> handler)
            {
                ListView innerListView = new ListView(Context);                
                innerListView.Adapter = new ArrayAdapter<String>(this._context, Android.Resource.Layout.SimpleListItem1, items);
                innerListView.ItemClick += (o, e) =>
                    {
                        handler.Invoke(o, new DialogClickEventArgs(e.Position));
                        this.DismissSafely();
                    };
                this.SetContentView(innerListView);
                return this;
            }        

            public StyleableAlertDialog SetItems(string[] items, IDialogInterfaceOnClickListener listener)
            {
                ListView innerListView = new ListView(Context);
                innerListView.Adapter = new ArrayAdapter<String>(this._context, Android.Resource.Layout.SimpleListItem1, items);
                innerListView.ItemClick += (o, e) =>
                {
                    listener.OnClick(this, e.Position);
                    this.DismissSafely();
                };
                this.SetContentView(innerListView);
                return this;
            }

            public new StyleableAlertDialog SetCancelable(bool flag)
            {
                base.SetCancelable(flag);
                return this;
            }

            /// <summary>
            /// Set custom content view for the dialog
            /// </summary>
            /// <param name="v">View to set</param>
            /// <returns></returns>
            public new StyleableAlertDialog SetContentView(View v)
            {
                _dialogView.SetContentView(v);
                return this;
            }

            public StyleableAlertDialog SetButtonFontSize(float sizeInSp)
            {
                _dialogView.SetButtonFontSize(sizeInSp);
                return this;
            }
    }
~~~~