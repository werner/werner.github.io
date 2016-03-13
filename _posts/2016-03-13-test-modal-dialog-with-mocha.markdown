---
layout: post
title: "Test a modal dialog in AngularJS with Mocha, Chai, Sinon and BardJS"
date: 2016-03-13 11:47:00 -0430
categories: angularjs modal test
---
I'm creating this post because I feel there's not much information about this subject online, so if anyone besides
me can use this, I can feel satisfied. I'll show how to test a bootstrap modal dialog box, using the resources 
provided by hottowel yeoman generator.

In the first place we need a way to create the modal, I choose a service in `app/common/bootstrap` path,
according to 
[bootstrap.dialog](https://github.com/johnpapa/HotTowel-Angular/blob/master/NuGet/HotTowel-NG/app/common/bootstrap/bootstrap.dialog.js), 
however I did a little bit change, I replaced the ModalInstance variable as a function
and use it as a controller in order to maintain consistency in all the application, like
[this](https://github.com/werner/angularjs_demo/blob/master/src/client/app/common/bootstrap/bootstrap.dialog.js#L11).

Now, we need to spy on the open modal method like this:
{% highlight javascript %}
sinon.spy($uibModal, 'open');
{% endhighlight %}

Then, wee need to stub the modal dialog, because we don't need to test the internals of this, we need to test 
the implementation we have in our application:
{% highlight javascript %}
modalInstance = {
    close: sinon.spy(),
    dismiss: sinon.spy(),
    result: {
        then: sinon.spy()
    }
};
{% endhighlight %}
As we can see in the code above we're stubbing the close, dismiss and then methods using spies,
so we can check when these methods are fired.

After that, we are ready to stub the parameters our ModalInstance controller
is using as `$scope, $uibModalInstance, options`:

{% highlight javascript %}
modalInstanceController = $controller('ModalInstance', {
    $scope: scope,
    $uibModalInstance: modalInstance,
    options: {
        title: 'Test title',
        message: 'Test message',
        okText: 'Ok',
        cancelText: 'Cancel'
    }
});
{% endhighlight %}

With all of the above, we can start testing the functions like `confirmationDialog` method,
checking at least the $uibModal open method is been called:
{% highlight javascript %}
it('should call confirmation dialog with arguments', function() {
    dialog.confirmationDialog('Test', 'a message', 'Accept', 'Cancel');
    expect($uibModal.open).to.have
        .been.called;
});
{% endhighlight %}

We can check the modalInstance close method is been called when the user press the ok button:
{% highlight javascript %}
it('should call ok action on dialog', function() {
    scope.ok();
    expect(modalInstance.close).to.have
        .been.called;
});
{% endhighlight %}

The full source code is available
[here](https://github.com/werner/angularjs_demo/blob/master/src/client/app/common/bootstrap/bootstrap.dialog.spec.js).
