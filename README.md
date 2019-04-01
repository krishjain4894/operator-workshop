# operator-workshop

This workshop will take you through creating a Kubernetes operator that will deploy an application and enure the version running is the one that matches our spec. Will will be able to configure the number of pods using a predefined container image available [here](https://quay.io/jharrington22). Our CRD will have 2 configurable fields, `replicas` and `appliactionVersion`. 

This operator will function the same way as a kubernetes [deployment](
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) does we will be able to do the following;
- Scale the number of pods up or down by changing the `replicas`
- Deploy a new image of our container by updating the `applicationVersion`

Although this example is very basic, this operator could be extend to manage all things that are related to your application, secrets, configmaps and services and pulling information required from external sources. The operator would own all the resources it creates which mans configuration of your application is very much simplified.

## Operator SDK Install
#### 1. Create a directory for our operator-sdk code
`mkdir -p $GOPATH/src/github.com/operator-framework`

`cd $GOPATH/src/github.com/operator-framework`

#### 2. Clone the Operator SDK
`git clone https://github.com/operator-framework/operator-sdk`

#### 3. Install deps
`make dep`
`make install`

#### 4. Check that its installed (if the command fails ensure you have $GOPATH in your $PATH)

```
operator-sdk --version
 operator-sdk version v0.6.0+git
```

## Building the Operator 

#### 1. Ensure that you have created a the required directory structure under $GOPATH for your repository which will house our new operator code inside a git repo
`mkdir $GOPATH/src/github.com/<YOUR_GITHUB_USERNAME>`
`cd $GOPATH/src/github.com/<YOUR_GITHUB_USERNAME>`

#### 2. Create our operator application-operator
`operator-sdk new application-operator`

#### 3. Lets add a new Custom Resource Definition (CRD) API which will recieve and store our Customer Resource (CR) changes. This will also create the basic configuration files we need to run the operator. 
`operator-sdk add api --api-version=application.examplle.com/v1alpha1 --kind=Application`

You should see that this command has also created the following manifests;
deploy/crds/
- Custom Resource Definition (application_v1alpha1_application_crd.yaml)
- Custom Resource (application_v1alpha1_application_cr.yaml)
deploy/
- RBAC Role (role.yml)
pkg/apis/application/v1alpha1/
API scaffolding, the most interesting file being for this workshop being;
- API types (application_types.go)

`git status` should show you our newly create and untracked files

#### 4. Adding to our newly created scaffold lets up date our API types file which and define the Spec and Status fields for our Custom Resource

```
type ApplicationSpec struct {
	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
	// Important: Run "operator-sdk generate k8s" to regenerate code after modifying this file
	// Add custom validation using kubebuilder tags: https://book.kubebuilder.io/beyond_basics/generating_crd.html
	Replicas           int  `json:"replicas"`
	ApplicationVersion string `json:"applicationVersion"`
}

// ApplicationStatus defines the observed state of Application
// +k8s:openapi-gen=true
type ApplicationStatus struct {
	// INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
	// Important: Run "operator-sdk generate k8s" to regenerate code after modifying this file
	// Add custom validation using kubebuilder tags: https://book.kubebuilder.io/beyond_basics/generating_crd.html
	Replicas           int  `json:"replicas"`
	ApplicationVersion string `json:"applicationVersion"`
}
```
#### 5. Commit our current status
`git add .`
`git commit -m "Added CRD and updated types"`

#### 6. Add our controller code
`operator-sdk add controller --api-version=application.example.com/v1alpha1 --kind=Application`

You should see that this command has created two files, the main file were interested in the the file containing the controller logic and recocile loop:

pkg/controller/application/application_controller.go

#### 7. Let create the logic that will deploy our application using a deployment!

