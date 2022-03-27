---
title: VueX Simple Explanation
date: 2021-06-30
description: VueX ~ an easy-to-understand explanation
---

## Introduction

Vue uses centralized storage to manage the state of all components of the application, and uses corresponding rules to ensure that the state changes in a predictable manner.

It solves the problem that multiple views rely on the same state and behaviors from different views need to change the same state, which lets us focus on the update of data rather than the transmission of data between components.

## Overview

- `State`: used for data storage, is the only data source in the store
  Getters: Like calculation attributes in vue, secondary packaging based on state data is often used for data filtering and correlation calculations of multiple data.

- `Mutations`: similar to functions, the only way to change state data, and cannot be used to handle **asynchronous events**.

- `Actions`: similar to mutations, used to submit mutations to change the state instead of directly changing the state, and can contain any **asynchronous operations**.

- `Modules`: similar to a namespace, used to define and operate the status of each module separately in the project, which is easy to maintain.

## State

It is a JS object to contain all states you will use in the whole project:

```js
// init data
const getDefaultState = () => {
  return {
    currentNav: "Home",
    serviceData: {},
    login: {
      clientType: "",
      token: "",
      cnum: "",
    },
  }
}

const state = getDefaultState()
```

## Mutations

If you want to change values in the state, you **MUST** use mutations to do that.

```js
const mutations = {
  setCurrentNav(state, current) {
    state.currentNav = current
  },
  setServiceData(state, current) {
    state.prepaidServiceData = current
  },
  setLogInStates(state, payload) {
    state.login[payload.key] = payload.value
  },
  resetState(state) {
    Object.assign(state, getDefaultState())
  },
}
```

A example to change the `currentNav` value:

```js
this.$store.commit("setCurrentNav", "profile")
```

use `commit` to call mutations. The first parameter is the name of mutations, the second parameter is the value for this state to update. I would like to call it 'payload' because you must pass the value to mutations to update.

## Actions

You **MUST** use the method in `actions` to handle asynchronous events and then use `commit` to update the value in `state`.

Here is an example how to get the data from api and update the state:

```js
const actions = {
  updateServiceData() {
    this.$axios
      .get("API/Account/GetTheServiceData")
      .then(response => {
        this.commit("setServiceData", JSON.parse(response.data))
      })
      .catch(error => {
        console.log(error)
      })
  },
}
```

## Modules

Vuex allows us to divide our store into modules. Each module can contain its own state, mutations, actions, getters, and even nested modules - it's fractal all the way down:

```js
const moduleStudents = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleTeachers = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    students: moduleStudents,
    teachers: moduleTeachers
  }
})
store.state.students // -> `moduleStudents`'s state
store.state.teachers // -> `moduleTeachers`'s state
```
