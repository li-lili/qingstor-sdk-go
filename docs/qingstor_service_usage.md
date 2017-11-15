# QingStor Service Usage Guide

Import the QingStor and initialize service with a config, and you are ready to use the initialized service. Service only contains one API, and it is "ListBuckets".
To use bucket related APIs, you need to initialize a bucket from service using "Bucket" function.

Each API function take a Input struct and return an Output struct. The Input struct consists of request params, request headers, request elements and request body, and the Output holds the HTTP status code, QingStor request ID, response headers, response elements, response body and error (if error occurred).

You can use a specified version of a service by import a service package with a date suffix.

``` go
import (
	// Import the latest version API
	qs "github.com/yunify/qingstor-sdk-go/service"
)
```

### Code Snippet

Initialize the QingStor service with a configuration

``` c++
QingStorService::initService(strConfigPath);	
```

List buckets

``` c++
//When list buckets
//Then list buckets status code is 200
QingStor::QsConfig qsConfig;
	qsConfig.loadConfigFile(strConfigPath);
	QingStorService qsService(qsConfig);
	
	ListBucketsInput input;
	ScenarioScope<ListBucketsOutput> contextOutput;
	//input.SetLocation("gd1");

	QsError err = qsService.listBuckets(input, *contextOutput);
	if (QsError::QS_ERR_NO_ERROR != err)
	{
		//context->result = (int)putObjectPropsOuput.GetResponseCode();
	}

	qsService.listBuckets(input, *contextOutput);
```

Initialize a QingStor bucket

``` c++
QingStorService::initService(strConfigPath);
	QingStor::QsConfig qsConfig;
	qsConfig.loadConfigFile(strConfigPath);

	ScenarioScope<TestBucketCtx> context;
	context->pQsService = new QingStorService(qsConfig);
	context->pQsBucket = new Bucket(qsConfig, "testmorvenhuang", "pek3a");
```

List objects in the bucket

``` c++
//When list objects
//Then list objects status code is 200
//And list objects keys count is 0
QingStor::QsConfig qsConfig;
	qsConfig.loadConfigFile(strConfigPath);
	QingStorService qsService(qsConfig);
	Bucket qsBucket = qsService.GetBucket("testmorvenhuang", "pek3a");

	ListObjectsInput input;
	ScenarioScope<ListObjectsOutput> contextOutput;

	QsError err = qsBucket.listObjects(input, *contextOutput);
```

PUT ACL of the bucket

``` c++
//Scenario: set the bucket ACL
//When put bucket ACL:
//		  """
//		  {
//			  "acl": [
//			  {
//				  "grantee": {
//					  "type": "group",
//						  "name" : "QS_ALL_USERS"
//				  },
//				  "permission" : "FULL_CONTROL"
//			  }
//			  ]
//		  }
//		  """
//Then put bucket ACL status code is 200
QingStor::QsConfig qsConfig;
	qsConfig.loadConfigFile(strConfigPath);
	QingStorService qsService(qsConfig);
	Bucket qsBucket = qsService.GetBucket("testmorvenhuang", "pek3a");

	PutBucketACLInput input;
	ScenarioScope<PutBucketACLOutput> contextOutput;

	std::vector<ACLType> aclList;
	ACLType acl;
	GranteeType grantee;

	grantee.SetType("group");
	grantee.SetName("QS_ALL_USERS");
	acl.SetGrantee(grantee);
	acl.SetPermission("FULL_CONTROL");

	aclList.push_back(acl);
	input.SetACL(aclList);
```

put object with key

``` c++
// When put object with key "<key>"
// Then put object status code is 201
ScenarioScope<TestObjectCtx> contextObjectTest;
	contextObjectTest->bucketName = "testmorvenhuang";

	QingStor::QsConfig qsConfig;
	qsConfig.loadConfigFile(strConfigPath);
	contextObjectTest->pQsBucket = new Bucket(qsConfig, "testmorvenhuang", "pek3a");
	Bucket qsBucket = *contextObjectTest->pQsBucket;

	PutObjectInput input;
	ScenarioScope<PutObjectOutput> contextOutput;


	std::shared_ptr<std::iostream> objectStream = std::make_shared<std::stringstream>();
	*objectStream << "thi is a test";
	objectStream->flush();
	input.SetBody(objectStream);
	input.SetContentLength(strlen("thi is a test"));
```
delete object with key

``` c++
//When delete object with key "<key>"
//Then delete object status code is 204
ScenarioScope<TestObjectCtx> contextObjectTest;

	Bucket qsBucket = *contextObjectTest->pQsBucket;

	DeleteObjectInput input;
	ScenarioScope<DeleteObjectOutput> contextOutput;

	QsError err = qsBucket.deleteObject(objectkey, input, *contextOutput);
//When delete the move object with key "<key>"
//Then delete the move object status code is 204
std::string objectkeyToDel = objectkey + "_move";

	ScenarioScope<TestObjectCtx> contextObjectTest;

	Bucket qsBucket = *contextObjectTest->pQsBucket;

	DeleteObjectInput input;
	ScenarioScope<DeleteObjectOutput> contextOutput;

	QsError err = qsBucket.deleteObject(objectkeyToDel, input, *contextOutput);
```

Initialize Multipart Upload

``` c++
//When initiate multipart upload with key "<key>"
//Then initiate multipart upload status code is 200
	ScenarioScope<TestListMultipartUploadsCtx> contextMultiPartObjectTest;
	contextMultiPartObjectTest->bucketName = "testmorvenhuang";

	QingStor::QsConfig qsConfig;
	qsConfig.loadConfigFile(strConfigPath);
	contextMultiPartObjectTest->pQsBucket = new Bucket(qsConfig, "testmorvenhuang", "pek3a");
	Bucket qsBucket = *contextMultiPartObjectTest->pQsBucket;

	InitiateMultipartUploadInput input;
	ScenarioScope<InitiateMultipartUploadOutput> contextOutput;

	//contextGiven->objectKey = objectkey;
```

Upload Multipart

``` c++
//When upload the first part with key "<key>"
//Then upload the first part status code is 201
	ScenarioScope<TestListMultipartUploadsCtx> contextMultiPartObjectTest;
	Bucket qsBucket = *contextMultiPartObjectTest->pQsBucket;

	UploadMultipartInput input;
	ScenarioScope<UploadMultipartOutput> contextOutput;

	std::shared_ptr<std::iostream> objectStream = std::make_shared<std::stringstream>();
	*objectStream << " |thi is a Part 1| ";
	objectStream->flush();
	input.SetBody(objectStream);
	input.SetContentLength(strlen(" |thi is a Part 1| "));
	input.SetPartNumber(1);
	input.SetUploadID(contextMultiPartObjectTest->uploadID);
//When upload the second part with key "<key>"
//Then upload the second part status code is 201
	ScenarioScope<TestListMultipartUploadsCtx> contextMultiPartObjectTest;
	Bucket qsBucket = *contextMultiPartObjectTest->pQsBucket;

	UploadMultipartInput input;
	ScenarioScope<UploadMultipartOutput> contextOutput;

	std::shared_ptr<std::iostream> objectStream = std::make_shared<std::stringstream>();
	*objectStream << " |thi is a Part 2| ";
	objectStream->flush();
	input.SetBody(objectStream);
	input.SetContentLength(strlen(" |thi is a Part 2| "));
	input.SetPartNumber(2);
	input.SetUploadID(contextMultiPartObjectTest->uploadID);

//When upload the third part with key "<key>"
//Then upload the third part status code is 201
	ScenarioScope<TestListMultipartUploadsCtx> contextMultiPartObjectTest;
	Bucket qsBucket = *contextMultiPartObjectTest->pQsBucket;

	UploadMultipartInput input;
	ScenarioScope<UploadMultipartOutput> contextOutput;

	std::shared_ptr<std::iostream> objectStream = std::make_shared<std::stringstream>();
	*objectStream << " |thi is a Part 3| ";
	objectStream->flush();
	input.SetBody(objectStream);
	input.SetContentLength(strlen(" |thi is a Part 3| "));
	input.SetPartNumber(3);
	input.SetUploadID(contextMultiPartObjectTest->uploadID);
```

Complete Multipart Upload

``` c++
//When complete multipart upload with key "<key>"
//Then complete multipart upload status code is 201
ScenarioScope<TestListMultipartUploadsCtx> contextMultiPartObjectTest;
	Bucket qsBucket = *contextMultiPartObjectTest->pQsBucket;

	CompleteMultipartUploadInput input;
	ScenarioScope<CompleteMultipartUploadOutput> contextOutput;
	input.SetUploadID(contextMultiPartObjectTest->uploadID);
```

Abort Multipart Upload

``` c++
//When abort multipart upload with key "<key>"
//Then abort multipart upload status code is 400
ScenarioScope<TestListMultipartUploadsCtx> contextMultiPartObjectTest;
	Bucket qsBucket = *contextMultiPartObjectTest->pQsBucket;

	AbortMultipartUploadInput input;
	ScenarioScope<AbortMultipartUploadOutput> contextOutput;
	input.SetUploadID(contextMultiPartObjectTest->uploadID);

	QsError err = qsBucket.abortMultipartUpload(objectkey, input, *contextOutput);
```
