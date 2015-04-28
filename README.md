# CPP-BSON
Header-only, based on mongodb BSON library

**CPP-BSON**
A BSON header only library based on MongoDB. Requires Boost header includes (no libs required).
Examples and tutorials from can be found [here.](https://github.com/mongodb/mongo-cxx-driver/wiki/Tutorial)

#How to save to JSON
```
#include <iostream>
#include <string>
#include <fstream>
#include <bson.h>

using namespace std;

int main() {
	bson::bob b;// same as mongo::BSONObjBuilder
	b.append("name", "Joe");
	b.append("age", 33);

	bson::bo p = b.obj(); // same as mongo::BSONObj

	std::ofstream fjson( "./out.json", std::ios::binary | std::ios::out | std::ios::trunc );
	fjson.clear();
	std::string jsonData = p.toString();
	fjson << jsonData;
	fjson.close();
}

```

#How to save to BSON
```
#include <iostream>
#include <string>
#include <fstream>
#include <bson.h>

using namespace std;

int main() {
	bson::bob b;// same as mongo::BSONObjBuilder
	b.append("name", "Joe");
	b.append("age", 33);

	bson::bo p = b.obj(); // same as mongo::BSONObj

	std::ofstream fbson( "./out.bson", std::ios::binary | std::ios::out | std::ios::trunc );
	fbson.clear();
	fbson.write(p.objdata(), p.objsize());
	fbson.close();
}
```
#How to read from BSON
```
#include <iostream>
#include <sstream>
#include <fstream>
#include <bson.h>

using namespace std;

int main() {
	std::ofstream fbson( "./src.bson", std::ios::binary | std::ios::in);
	stringstream buffer;
	buffer << fbson.rdbuf();
	fbson.close();
	auto data = buffer.str(); //save file contents into string

	bson::bo b(data.c_str()); // same as mongo::BSONObj
	if( b.hasElement("name") && b.hasElement("age") ) {
		cout << b["name"] << " : " << b["age"];
	}
	cin.get();
}
```
#Another way to create a BSON object
```
#include <iostream>
#include <bson.h>
using namespace std;

int main() {
	bson::bo b = BSON( "name" << "Joe" << "age" << 33 );
	if( b.hasElement("name") && b.hasElement("age") ) {
		cout << b["name"] << " : " << b["age"];
	}
	
	cin.get();
}
```
#Create a BSON array and save it into file
```
#include <iostream>
#include <fstream>
#include <bson.h>

using namespace std;

int main() {
	//Create data
	bson::bob firstElement;
	firstElement.append("name", "Joe");
	firstElement.append("age", 33);

	bson::bob lastElement;
	lastElement.append("name", "Jack");
	lastElement.append("age", 16);

	//Create array
	mongo::BSONArrayBuilder array;
	array.append(firstElement.obj());
	array.append(lastElement.obj());
	bson::bo serializedArray = array.arr();

	//Add array into more general object
	bson::bob container;
	container.appendArray("elements", serializedArray);

	//Fill container array
	bson::bo p = container.obj(); // same as mongo::BSONObj

	//Save array
	ofstream fbson("./out.bson", std::ios::binary | std::ios::out | std::ios::trunc);
	fbson.clear();
	fbson.write(p.objdata(), p.objsize());
	fbson.close();
}

```
#Read a BSON array from file
```
#include <iostream>
#include <sstream>
#include <fstream>
#include <bson.h>

using namespace std;

int main() {
	std::ofstream fbson( "./in.bson", std::ios::binary | std::ios::in);
	stringstream buffer;
	buffer << fbson.rdbuf();
	fbson.close();
	auto data = buffer.str(); //save file contents into string

	bson::bo b(data.c_str());
	auto name = "elements";
	bson::be elements = b.getFieldDottedOrArray(name);

	for(auto & element : elements.Array()) {
		bson::bo data = element.Obj();
		if( data.hasElement("name") && data.hasElement("age") ) {
			cout << data["name"] << " : " << data["age"] << endl;
		}
	}

	cin.get();
}
```