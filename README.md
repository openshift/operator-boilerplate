# Operator Boilerplate (Legacy)

Operator Boilerplate is a tiny library containing a few helpers to make writing an operator or custom controller easier by abstracting some of the repetitive mechanisms common to most controllers.

## Install

    go get github.com/openshift/operator-boilerplate

## Use

The pattern for use is something like this:

```golang
// starter.go

runner := example.NewExampleController(
   // pass in informers, clients, etc needed to create your controller
)

go runner.Run(ctx.Done())


// example_controller.go
import "github.com/openshift/operator-boilerplate/openshift/operator"

type ExampleController struct {
    // clients,
    // informers,
    // etc
}

func NewExampleController(/* client,etc */) operator.Runner {
   c := ExampleController{
      // client, etc
   }

   optionalFilter :=operator.FilterByNames("example")

   return operator.New("example-controller", c,
      operator.WithInformer(someInformer.Resource(), optionalFilter),
      // other filters
   )
}

func(c *ExampleController) Key(metav1.Object, error) {
  // the resource most important to your controller can act as the key
  return c.exampleClient.Get("example", metav1.GetOptions{})
}

func(c *ExampleController) Sync(obj metav1.Object) error {
    startTime := time.Now()
	klog.V(4).Infof("started syncing operator %q (%v)", obj.GetName(), startTime)
	defer klog.V(4).Infof("finished syncing operator %q (%v)", obj.GetName(), time.Since(startTime))

    // cast the key to the appropriate resource type
    exampleConfig := obj.(*examplev1.Example)

    if err := c.handleSync(exampleConfig); err != nil {
		return err
	}

	return nil
}


func(c *ExampleController) handleSync(config *examplev1.Example) {
    // if the controller responds to management state:
	switch config.Spec.ManagementState {
	case operatorsv1.Managed:
		klog.V(4).Infoln("example is in a managed state: syncing resources")
	case operatorsv1.Unmanaged:
		klog.V(4).Infoln("example is in an unmanaged state: skipping sync")
		return nil
	case operatorsv1.Removed:
		klog.V(4).Infoln("example is in a removed state: deleting resources")
		return c.deleteExampleResources()
	default:
		return fmt.Errorf("example is in an unknown state: %v", updatedOperatorConfig.Spec.ManagementState)
	}

    // if passed management state checks, handle sync loop here
    err := c.SyncExampleResources(config)
    // update your aggregated status
    HandleDegraded(c.exampleConfigClient, "ExampleType", "ExampleReason", true||false)

    // bubble up errors
    if err != nil {
       return err
    }
}

```



## Existing

There are several operators currently using this library:

- [OpenShift Cluster Console Operator](https://github.com/openshift/console-operator)
- [OpenShift Cluster Authentication Operator](https://github.com/openshift/cluster-authentication-operator)
- [OpenShift Service CA Operator](https://github.com/openshift/service-ca-operator)
