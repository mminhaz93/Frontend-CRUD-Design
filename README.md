# Frontend CRUD Design

### Objectives

After this lesson, you should be able to:

- recognize the structure of a full CRUD front-end client
- describe the importance of the front-end in a RESTful web application
- be able to connect together all the different components of a full CRUD client

## RESTful Web Apps

One of the qualifications for a RESTful web app is that the client is separate from the back-end server in order keep concerns separated.

1. The two must work independently from each other. The client could be swapped out for a different front-end and vice versa.
2. The backend endpoints must follow a uniform pattern (i.e. GET /projects, POST /projects, etc).
3. Also the back-end must be "stateless". It shouldn't store any information about the requests it receives and each request should be treated as a brand new independent request.
4. The frontend must be able to cache the data it receives to improve performance. Have you ever received a network response of `304` when making repeated requests? The `304` status code stands for "not modified" and means that the data if identical to a previously cached response.
5. Also the frontend must not have any idea of what's going on in the back-end. The endpoint it hits could be an API that is accessing the database _or_ it could just be chaining to another API to get additional data. The whole idea is that each layer works independently from each other and is unaware of what the other layers do.

## Starting the Front-end

If we already have our routes for our API, or if we are assuming that the routes follow convention, we have everything we need to build out our axios calls. Let's take a look at how those should look:

Read All:

```
export const getAllItems = async () => {
  const resp = await axios.get(`${BASE_URL}/items`);
  return resp.data;
};
```

Notice how we don't need any more data than the base url and the url path for our endpoint. In this example, we can assume that the `BASE_URL` variable was set to the base url that we want to hit.

Read One:

```
export const getOneItem = async (id) => {
  const resp = await axios.get(`${BASE_URL}/items/${id}`);
  return resp.data;
}
```

This API call looks very similar to our previous one except that it takes the `id` of the item that we want to get. We don't need to worry about what the id's value is in this function. Our only concern is to pass it to the path of our endpoint as a param. Separation of concerns!

Delete/Destroy:

```
export const destroyItem = async (id) => {
  const resp = await axios.delete(`${BASE_URL}/items/${id}`);
  return resp;
}
```

At a glance, it almost looks like we didn't change anything here. The only difference is the axios method is now delete instead of get. We still pass in the id of the item that we are referring to. Additionally since there may not be data in our response, we only need to return `resp`. It might have an error message that we would want to handle so we still return it. (Also, the function name is different, in case you're copy/pasting your previous function).


Create:

```
export const postItem = async (itemData) => {
  const resp = await axios.post(`${BASE_URL}/items`, itemData);
  return resp.data;
}
```

This time, we are passing in data for the item that we want to store in the database. However, we don't pass it as a param; Axios `POST` and `PUT` methods can take a second argument, which will be the body of our request.

Update:

```
export const putItem = async (id, updatedData) => {
  const resp = await axios.put(`${BASE_URL}/items/${id}`, updatedData);
  return resp.data;
}
```

Here we have both the item's id and the updated data for that item. It's almost a combination of the `create` and `getOne`/`delete` API calls.

## Component Structure

So what's next? We don't have much in our app right now outside of these API calls that we're not using. This is where having nice wireframes comes in handy. For a larger project, the more comprehensive our wireframes are, the easier it will be to build out our components. But for right now, let's take a look at a simple component tree:

![component-tree](./component-tree.png)

It looks like all four of our components are being called in app. This doesn't mean that they are different views though. We could set up multiple views using react router and have them all called in app. But if we wanted to have the create form at the top of our item list, we could call them like this:

```
<Route path='/' render={() => (
  <>
    <CreateForm />
    <ItemList />
  </>
)} />
```

We still have some freedom over how we want to display the components even when we import them all into app. This also makes our component tree very simple. Unless the design requires something more complex, keeping it simple is pretty nice and easy to maintain.

## Function/Data Placement 

What about our data? Where should we store state? Also where do our methods go? Keeping everything centralized in `App` is an easy approach to this. Later, when we are much more comfortable with web development, it'll be easy to get the feel for where exactly our values need to live. That also requires a good deal of planning before starting a project.

When we have our methods in `App`, we simply need to pass them down to our child components to be able to call them when we need them. Here's a diagram of how functions flow typically in a CRUD app:

![function flow](./React-CRUD-functional-flow.png)

There are a lot of moving parts here. If we don't need to have a separate view for a single item, we can just keep our "edit" and "delete" button on the items in our item list. Them we could remove the `SingleItem` component from this list.

### Read

Looking at this diagram, we see that `read` looks pretty simple. It's a function in api-helper that makes our axios call and another function in App that sets state. We could call our setState function in `componentDidMount` if we wanted our data to load with the page.

### Create

Create is the same but it needs to get data from the form first. Also, it isn't triggered until the form is submitted.

### Delete

Delete doesn't need form data, but it does need the id of a single item. We can get this from the `.map` inside of our item list or from the individual item view, if we have it. We can trigger this function with the onClick of a button.

### Update

Update has the most moving parts here, but it's really just a combination of delete and create. We start with an onClick in the item list or single item page view. We need to grab the entire item, including its id and save it in state in App. We'll need our id once the update form is submitted. We need to replace our empty form data in state with the rest of the items data. If we redirect the user to the update form now, they should be looking at a form with the items already in it, ready to be updated. From here, it's the same as create. On submit, the form data is sent to our API call and then set in state of our app.

The only function that we are missing from the diagram is for our controlled component: `handleChange`. That should live in App since that is where the state for our form data is. As for `handleSubmit`, our setState functions for create and update are our handleSubmit functions.

## Setting State

We have four different setState functions in our app and they all need to behave differently. Let's look at how they should look and see what they are doing.

### Read

```
this.setState({ items });
```

This one is pretty simple since it only needs to set up our initial state.

### Create

```
this.setState(prevState => ({
  items: [...prevState.items, newItem]
}))
```

State is immutable and we can't change it. However we _can_ replace state with something new. This is where `prevState` comes in handy. In this `setState`, we are replacing `items` with a new array. Inside that array, we spread out all the contents of the items that were in state previously and add our new item.

### Update

```
this.setState(prevState => ({
  items: prevState.items.map(item => item.id === updatedId ? newItem : item)
})
```

Update again seems like it has a lot going on but it's only three different pieces that we're all familiar with.

1. First, we setState with prevState.
2. Next, we map through our previous items. remember, `.map` returns a new array so we are still replacing state and not mutating it.
3. inside our map, we have a ternary. It is checking for the item id that matches the id of the item we updated. For that one item, we return our updated item from our api call. For all other items, we return the previous item.

### Delete

```
this.setState(prevState => ({
  items: prevState.item.filter(item => item.id !== deletedId)
})
```

Delete is pretty similar to update here except that we are filtering out our deleted item. We only want the items that don't match the id of the one we deleted.

## Lab Practice

I know that we've already made react apps in the past, but never with this many moving parts. Let's get some practice putting these pieces together.
