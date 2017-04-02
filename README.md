# Firebase-JSON

### Suppose we already have data in our database

Creating data reference
```javascript
// To work with data in firebase database, we need to first create database reference
// this can be on the root node or a particular node
// for the root node, simply type
const dataRef = firebase.database().ref();
// selecting the 'users' child node in the database
const usersRef = firebase.database().ref().child('users');
// if we want to further select a particular user nested under the 'users' node, 
// we can do so by selecting from the usersRef
const user_01 = userRef.child('01');
```

Fetching existing data from a particular user node

```javascript
// to simply fecth and display the data once, call the once function
// 'value' event reads data at a path
user_01.once('value', snap => {
    demo.innerText = JSON.stringify(snap.val(), null, 3);
});

// when being used with on() methods, 'value' event listens to the changes at a path 
// and provides real time data change
// any changes at the user_01 path will be displayed at the html element with the demo id
// without refreshing the page
user_01.on('value', snap => {
    demo.innerText = JSON.stringify(snap.val(), null, 3);
});

// Another way to write the callback function
function show_data(snapshot) {
    var value = snapshot.val();
    demo.innerText = JSON.stringify(value, null, 3);
}
user_01.once('value', show_data);
```

Updating existing data at a particular node
```javascript
// suppose we only want to update a particular information of user_01
// say his address

// code that does not work
// create a variable to later store the user's info fetched from firebase database
var user_01_value;
user_01.once('value', snap => {
    // assign the value at user_01 path to user_01_value
    user_01_value = snap.val();
});

// get the id, which we use as the key to reference the user in firebase
var data_key = user_01_value.id; 
user_01_value.address = "New Address";
var update = {}; // to update infomation, we need to create a empty JSON
// use data_key and the actually value to set the update JSON object
update[data_key] = user_01_value; 
// call the update() method with the update JSON object on usersRef to update its child nodes
usersRef.update(update);

// The problem here is that since Firebase works with AJAX, 
// the callback function takes a while to actually get
// the result from Firebase. Meanwhile, JavaScript keeps on running the sequential code. 
// Since no actual value has been assigned to user_01_value, user_01_value remains undefined. 
// Thus, user_01_value.id will cause JavaScript to throw an error.

// What we can do to update
// update() takes the 'node' that we want to update, for example, 'users'
// and 'key' to a particular node, and the value associated with that 'key'
function update(node, key, value) {
    var ref = firebase.database().ref();
    var obj = {};
    obj[key] = value;
    ref.child(node).update(obj).then(function() {
        console.log('Updated...');
    })
}

// note that we can refer to the child node of a node like this
// 'users/01/address'
// then we can update uesr_01's address by simply calling the update function
update('users/01', 'address', 'New address');
```

Removing a particular node
```javascript
// simply calling the remove() mothod a node reference will do the job
user_01.remove();
```

### To actually save data to our database
Saving data to Firebase database
```javascript
var user_02_data = {
    id: '02',
    age: 21,
    address: "1 University Avenue",
    hobbies: {
        music: "Eminem",
        sport: "Football"
    },
    name: "Leo"
}

// since we want to store user_02's info under the 'users' node,
// we call child() from userRef to refer to usersRef's child nodes.
// Then use the id of user_02 as the the name of child node of 'users', 
// which can be thought of as the name of the 'key' in a JSON object.
// If the name of the child node already exists, then set() method 
// will overwrite the data at that node. 
// Otherwise, set() mothod will create a new node and set the correspoding data.
usersRef.child(user_02_data.id).set(user_02_data);
```
