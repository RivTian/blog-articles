
![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/0jxLXp.jpg)

This article will show you how to utilize Microsoft Visual Studio to compile and install the [MongoDB C](https://www.mongodb.com/docs/drivers/c/) and [C++ drivers](https://www.mongodb.com/docs/drivers/cxx/) on Windows, and use these drivers to create a console application that can interact with your MongoDB data by performing basic [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations.

Tools and libraries used in this tutorial:

Tools and libraries used in this tutorial:

1. Microsoft Windows 11
    
2. Microsoft Visual Studio 2022 17.3.6
    
3. Language standard: C++17
    
4. MongoDB C Driver version: 1.23
    
5. MongoDB C++ Driver version: 3.7.0
    
6. boost: 1.80.0
    
7. Python: 3.10
    
8. CMake: 3.25.0
    
#### Prerequisites

1. [MongoDB Atlas account](https://www.mongodb.com/developer/products/atlas/free-atlas-cluster/) with a cluster created.
    
2. _(Optional)_ [Sample dataset](https://www.mongodb.com/developer/products/atlas/atlas-sample-datasets/) loaded into the Atlas cluster.
    
3. Your machine’s IP address is [whitelisted](https://www.mongodb.com/docs/guides/atlas/network-connections/). Note: You can add _0.0.0.0/0_ as the IP address, which should allow access from any machine. This setting is not recommended for production use.
    

[](https://www.mongodb.com/developer/products/mongodb/getting-started-mongodb-cpp/#installation--ide-and-tools)

#### Installation: IDE and tools

Step 1: Install Visual Studio: [Download Visual Studio Tools - Install Free for Windows, Mac, Linux](https://visualstudio.microsoft.com/downloads/).

In the Workloads tab during installation, select “Desktop development with C++.”

![Microsoft Visual Studio installation](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/zjGAWi.jpg)


Step 2: Install CMake: [Download | CMake](https://cmake.org/download/)

- For simplicity, choose the installer.
    
- In the setup, make sure to select “Add CMake to the system PATH for all users.” This enables the CMake executable to be easily accessible.

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/07KZTr.jpg)

Step 3: Install Python 3: [Download Python](https://www.python.org/downloads/).

Step 4: _(Optional)_ Download boost library from [Boost Downloads](https://www.boost.org/users/download/) and extract it to _C:\boost_.

#### Installation: Drivers

Detailed instructions and configurations available here:

- [Installing the mongocxx driver](https://mongocxx.org/mongocxx-v3/installation/)
    
- [Installing the MongoDB C Driver (libmongoc) and BSON library (libbson)](http://mongoc.org/libmongoc/current/installing.html)
    

[](https://www.mongodb.com/developer/products/mongodb/getting-started-mongodb-cpp/#step-1--install-c-driver)

##### Step 1: Install C Driver

C++ Driver has a dependency on C driver. Hence, we need to install C Driver first.

- Download C Driver
    
    - Check compatibility at [Windows - Installing the mongocxx driver](https://mongocxx.org/mongocxx-v3/installation/windows/) for the driver to download.
        
    - Download release tarball — [Releases · mongodb/mongo-c-driver](https://github.com/mongodb/mongo-c-driver/releases) — and extract it to _C:\Repos\mongo-c-driver-1.23.0_.
        
    
- Setup build via CMake
    
    - Launch powershell/terminal as an administrator.
        
    - Navigate to _C:\Repos\mongo-c-driver-1.23.0_ and create a new folder named _cmake-build_ for the build files.
        
    - Navigate to _C: \Repos\mongo-c-driver-1.23.0\cmake-build_.
        
    - Run the below command to configure and generate build files using CMake.
        
    

Code Snippet

```bash
cmake -G "Visual Studio 17 2022" -A x64 -S "C:\Repos\mongo-c-driver-1.23.0" -B "C:\Repos\mongo-c-driver-1.23.0\cmake-build"
```


![Setting up MongoDB C driver build via CMake and Microsoft Visual Studio](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/PsrP9O.jpg)

Note: Build setup can be done with the CMake GUI application, as well.

- Execute build
    
    - Visual Studio’s default build type is Debug. A release build with debug info is recommended for production use.
        
    - Run the below command to build and install the driver
        
    

```bash
cmake --build . --config RelWithDebInfo --target install
```


![Building MongoDB C driver via CMake and Microsoft Visual Studio](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/N37DvC.jpg)

- You should now see libmongoc and libbson installed in _C:/Program Files/mongo-c-driver_.
    

![Building MongoDB C driver via CMake and Microsoft Visual Studio](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/yvWXoq.jpg)

- Move the _mongo-c-driver_ to _C:/_ for convenience. Hence, C Driver should now be present at _C:/mongo-c-driver_.

##### Step 2: Install C++ Driver

- Download C++ Driver
    
    - Download release tarball — [Releases · mongodb/mongo-cxx-driver](https://github.com/mongodb/mongo-cxx-driver/releases) — and extract it to _C:\Repos\mongo-cxx-driver-r3.7.0_.
        
    
- Set up build via CMake
    
    - Launch powershell/terminal as an administrator.
        
    - Navigate to _C:\Repos\mongo-cxx-driver-r3.7.0\build_.
        
    - Run the below command to generate and configure build files via CMake.
        
    

```bash
cmake .. -G "Visual Studio 17 2022" -A x64 -DCMAKE_CXX_STANDARD=17 -DCMAKE_CXX_FLAGS="/Zc:__cplusplus /EHsc" -DCMAKE_PREFIX_PATH=C:\mongo-c-driver -DCMAKE_INSTALL_PREFIX=C:\mongo-cxx-driver
```


![Setting up MongoDB C++ driver build via CMake and Microsoft Visual Studio](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/P5ZWag.jpg)

Note: Setting _DCMAKE_CXX_FLAGS_ should not be required for C++ driver version 3.7.1 and above.

- Execute build
    
    - Run the below command to build and install the driver
        
    

```bash
cmake --build . --config RelWithDebInfo --target install
```


- You should now see C++ driver installed in _C:\mongo-cxx-driver_.
    

#### Visual Studio: Setting up the dev environment

- Create a new project in Visual Studio.
    
- Select _Console App_ in the templates.
    

![New project options in Microsoft Visual Studio](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/RjJsuO.jpg)

- Visual Studio should create a new project and open a .cpp file which prints “Hello World.” Navigate to the Solution Explorer panel, right-click on the solution name (_MongoCXXGettingStarted_, in this case), and click Properties.
    

![Microsoft Visual Studio solution properties](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/4UlnoM.jpg)

- Go to _Configuration Properties > C/C++ > General > Additional Include Directories_ and add the include directories from the C and C++ driver installation folders, as shown below.
    

![Microsoft Visual Studio include directories](https://www.mongodb.com/developer/_next/image/?url=https%3A%2F%2Fmongodb-devhub-cms.s3.us-west-1.amazonaws.com%2Finclude_directories_da1b8bf333.png&w=3840&q=75)

- Go to _Configuration Properties > C/C++ > Language_ and change the _C++ Language Standard_ to C++17.
    

![Microsoft Visual Studio C++ language standard](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/XhAQRr.jpg)

- Go to _Configuration Properties > C/C++ > Command Line_and add _/Zc:__cplusplus_ in the _Additional Options_ field. This flag is needed to opt into the [correct definition of __cplusplus](https://devblogs.microsoft.com/cppblog/msvc-now-correctly-reports-__cplusplus/).
    
- Go to _Configuration Properties > Linker > Input_ and add the driver libs in _Additional Dependencies_ section, as shown below.
    

![Microsoft Visual Studio linker dependencies](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/6cImEx.jpg)

- Go to _Configuration Properties > Debugging > Environment_to add a path to the driver executables, as shown below.
    

![Microsoft Visual Studio debugging environment](https://www.mongodb.com/developer/_next/image/?url=https%3A%2F%2Fmongodb-devhub-cms.s3.us-west-1.amazonaws.com%2Fdebugging_environment_6de4cc98b8.png&w=3840&q=75)

#### Building the console application

Source available [here](https://github.com/mongodb-developer/get-started-cxx/blob/main/student-records-console-app/studentRecordsConsoleApp.cpp)

Let’s build an application that maintains student records. We will input student data from the user, save them in the database, and perform different CRUD operations on the database.

##### Connecting to the database

Let’s start with a simple program to connect to the MongoDB Atlas cluster and access the databases. [Get the connection string](https://www.mongodb.com/docs/guides/atlas/connection-string/) (URI) to the cluster and create a new environment variable with key as _“MONGODB_URI”_ and value as the connection string (URI). It’s a good practice to keep the connection string decoupled from the code.

Tip: Restart your machine after creating the environment variable in case the _“getEnvironmentVariable”_ function fails to retrieve the environment variable.

```cpp
#include <mongocxx/client.hpp>
#include <bsoncxx/builder/stream/document.hpp>
#include <bsoncxx/json.hpp>
#include <mongocxx/uri.hpp>
#include <mongocxx/instance.hpp>
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;
std::string getEnvironmentVariable(std::string environmentVarKey)
{
	char* pBuffer = nullptr;
	size_t size = 0;
	auto key = environmentVarKey.c_str();
	// Use the secure version of getenv, ie. _dupenv_s to fetch environment variable.
	if (_dupenv_s(&pBuffer, &size, key) == 0 && pBuffer != nullptr)
	{
		std::string environmentVarValue(pBuffer);
		free(pBuffer);
		return environmentVarValue;
	}
	else
	{
		return "";
	}
}
auto mongoURIStr = getEnvironmentVariable("MONGODB_URI");
static const mongocxx::uri mongoURI = mongocxx::uri{ mongoURIStr };
// Get all the databases from a given client.
vector<string> getDatabases(mongocxx::client& client)
{
	return client.list_database_names();
}
int main()
{
    // Create an instance.
    mongocxx::instance inst{};
    mongocxx::options::client client_options;
    auto api = mongocxx::options::server_api{ mongocxx::options::server_api::version::k_version_1 };
    client_options.server_api_opts(api);
    mongocxx::client conn{ mongoURI, client_options}
     auto dbs = getDatabases(conn);
     for (auto db : dbs)
     {
         cout << db << endl;
      }
      return 0;
}
```

Click on “Launch Debugger” to launch the console application. The output should looks something like this:

![Microsoft Visual Studio debug console](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/RKRhkN.jpg)



##### CRUD operations

[Full tutorial](https://mongocxx.org/mongocxx-v3/tutorial/)

Since the database is successfully connected to our application, let’s write some helper functions to interact with the database, performing CRUD operations.
###### Create

```cpp
// Create a new collection in the given database.
void createCollection(mongocxx::database& db, const string& collectionName)
{
	db.create_collection(collectionName);
}
// Create a document from the given key-value pairs.
bsoncxx::document::value createDocument(const vector<pair<string, string>>& keyValues)
{
	bsoncxx::builder::stream::document document{};
	for (auto& keyValue : keyValues)
	{
		document << keyValue.first << keyValue.second;
	}
	return document << bsoncxx::builder::stream::finalize;
}
// Insert a document into the given collection.
void insertDocument(mongocxx::collection& collection, const bsoncxx::document::value& document)
{
	collection.insert_one(document.view());
}
```

###### Read

```cpp
// Print the contents of the given collection.
void printCollection(mongocxx::collection& collection)
{
	// Check if collection is empty.
	if (collection.count_documents({}) == 0)
	{
		cout << "Collection is empty." << endl;
		return;
	}
	auto cursor = collection.find({});
	for (auto&& doc : cursor)
	{
		cout << bsoncxx::to_json(doc) << endl;
	}
}
// Find the document with given key-value pair.
void findDocument(mongocxx::collection& collection, const string& key, const string& value)
{
	// Create the query filter
	auto filter = bsoncxx::builder::stream::document{} << key << value << bsoncxx::builder::stream::finalize;
	//Add query filter argument in find
	auto cursor = collection.find({ filter });
	for (auto&& doc : cursor)
	{
          cout << bsoncxx::to_json(doc) << endl;
	}
}
```

###### Update

```cpp
// Update the document with given key-value pair.
void updateDocument(mongocxx::collection& collection, const string& key, const string& value, const string& newKey, const string& newValue)
{
	collection.update_one(bsoncxx::builder::stream::document{} << key << value << bsoncxx::builder::stream::finalize,
		bsoncxx::builder::stream::document{} << "$set" << bsoncxx::builder::stream::open_document << newKey << newValue << bsoncxx::builder::stream::close_document << bsoncxx::builder::stream::finalize);
}
```
###### Delete

```cpp
// Delete a document from a given collection.
void deleteDocument(mongocxx::collection& collection, const bsoncxx::document::value& document)
{
	collection.delete_one(document.view());
}
```

##### The main() function

With all the helper functions in place, let’s create a menu in the main function which we can use to interact with the application.

```cpp
// ********************************************** I/O Methods **********************************************
// Input student record.
void inputStudentRecord(mongocxx::collection& collection)
{
	string name, rollNo, branch, year;
	cout << "Enter name: ";
	cin >> name;
	cout << "Enter roll number: ";
	cin >> rollNo;
	cout << "Enter branch: ";
	cin >> branch;
	cout << "Enter year: ";
	cin >> year;
	insertDocument(collection, createDocument({ {"name", name}, {"rollNo", rollNo}, {"branch", branch}, {"year", year} }));
}
// Update student record.
void updateStudentRecord(mongocxx::collection& collection)
{
	string rollNo, newBranch, newYear;
	cout << "Enter roll number: ";
	cin >> rollNo;
	cout << "Enter new branch: ";
	cin >> newBranch;
	cout << "Enter new year: ";
	cin >> newYear;
	updateDocument(collection, "rollNo", rollNo, "branch", newBranch);
	updateDocument(collection, "rollNo", rollNo, "year", newYear);
}
// Find student record.
void findStudentRecord(mongocxx::collection& collection)
{
	string rollNo;
	cout << "Enter roll number: ";
	cin >> rollNo;
	findDocument(collection, "rollNo", rollNo);
}
// Delete student record.
void deleteStudentRecord(mongocxx::collection& collection)
{
	string rollNo;
	cout << "Enter roll number: ";
	cin >> rollNo;
	deleteDocument(collection, createDocument({ {"rollNo", rollNo} }));
}
// Print student records.
void printStudentRecords(mongocxx::collection& collection)
{
	printCollection(collection);
}
// ********************************************** Main **********************************************
int main()
{
     if(mongoURI.to_string().empty())
    {
	cout << "URI is empty";
	return 0;
    }
    // Create an instance.
    mongocxx::instance inst{};
    mongocxx::options::client client_options;
    auto api = mongocxx::options::server_api{ mongocxx::options::server_api::version::k_version_1 };
    client_options.server_api_opts(api);
    mongocxx::client conn{ mongoURI, client_options};
	const string dbName = "StudentRecords";
	const string collName = "StudentCollection";
	auto dbs = getDatabases(conn);
	// Check if database already exists.
	if (!(std::find(dbs.begin(), dbs.end(), dbName) != dbs.end()))
	{
		// Create a new database & collection for students.
conn[dbName];
	}
	auto studentDB = conn.database(dbName);
	auto allCollections = studentDB.list_collection_names();
	// Check if collection already exists.
	if (!(std::find(allCollections.begin(), allCollections.end(), collName) != allCollections.end()))
	{
		createCollection(studentDB, collName);
	}
	auto studentCollection = studentDB.collection(collName);
	// Create a menu for user interaction
	 int choice = -1;
	 do while (choice != 0)
	 {
		 //system("cls");
		 cout << endl << "**************************************************************************************************************" << endl;
		 cout << "Enter 1 to input student record" << endl;
		 cout << "Enter 2 to update student record" << endl;
		 cout << "Enter 3 to find student record" << endl;
		 cout << "Enter 4 to delete student record" << endl;
		 cout << "Enter 5 to print all student records" << endl;
		 cout << "Enter 0 to exit" << endl;
		 cout << "Enter Choice : "; 
		 cin >> choice;
		 cout << endl;
		 switch (choice)
		 {
		 case 1:
			 inputStudentRecord(studentCollection);
			 break;
		 case 2:
			 updateStudentRecord(studentCollection);
			 break;
		 case 3:
			 findStudentRecord(studentCollection);
			 break;
		 case 4:
			 deleteStudentRecord(studentCollection);
			 break;
		 case 5:
			 printStudentRecords(studentCollection);
			 break;
		 case 0:
			 break;
		 default:
			 cout << "Invalid choice" << endl;
			 break;
		 }
	 } while (choice != 0);
	 return 0;
}
```

#### Application in action

When this application is executed, you can manage the student records via the console interface. Here’s a demo:

https://youtu.be/JsHdysx8yyw

You can also see the collection in Atlas reflecting any change made via the console application.

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/Z8AXr1.jpg)

#### Wrapping up

With this article, we covered installation of C/C++ driver and creating a console application in Visual Studio that connects to MongoDB Atlas to perform basic CRUD operations.

More information about the C++ driver is available at [MongoDB C++ Driver](https://www.mongodb.com/docs/drivers/cxx/).