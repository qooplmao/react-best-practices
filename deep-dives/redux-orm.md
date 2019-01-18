# redux-orm

This is a repository for my own understanding of redux-orm and Redux structuring in general. I will be using this to document my notes while reading:  

[Practical Redux](https://blog.isquaredsoftware.com/series/practical-redux/)

[Practical Redux Training Course](https://blog.isquaredsoftware.com/2017/11/practical-redux-educative-course/)

**Assumed knowledge:** ORM Libraries (at least the concept) and relational table schemas

## Basics

### Define your model classes

```javascript
// models.js
import {Model, fk, oneToOne, many} from "redux-orm";

export class Pilot extends Model{}
Pilot.modelName = "Pilot";
Pilot.fields = {
  mech : fk("Battlemech"),
  lance : oneToOne("Lance")
};

export class Battlemech extends Model{}
Battlemech.modelName = "Battlemech";
Battlemech.fields = {
    pilot : fk("Pilot"),
    lance : oneToOne("Lance"),
};

export class Lance extends Model{}
Lance.modelName = "Lance";
Lance.fields = {
    mechs : many("Battlemech"),
    pilots : many("Pilot")
}
```

### Register your Schema

```javascript
import {Schema} from "redux-orm";
import {Pilot, Battlemech, Lance} from "./models";

const schema = new Schema();
schema.register(Pilot, Battlemech, Lance);
export default schema;
```

### Set up Store and Reducer

```javascript
// entitiesReducer.js
import schema from "models/schema";

// This gives us a set of "tables" for our data, with the right structure
const initialState = schema.getDefaultState();

export default function entitiesReducer(state = initialState, action) {
    switch(action.type) {
        case "PILOT_CREATE": {
            const session = schema.from(state);
            const {Pilot} = session;

            // Queue up a "creation" action inside of Redux-ORM
            const pilot = Pilot.create(action.payload.pilotDetails);

            // Applies the queued actions and returns an updated
            // "tables" structure, with all updates handled immutably
            return session.reduce();
        }
        // Other actual action cases would go here
        default : return state;
    }
}

// rootReducer.js
import {combineReducers} from "redux";
import entitiesReducer from "./entitiesReducer";

const rootReducer = combineReducers({
    entities: entitiesReducer
});

export default rootReducer;
```