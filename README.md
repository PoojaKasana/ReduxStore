# ReduxStore
This is the implementation of redux store. One advantage of this kind of store is that even if you reload the webpage, the store will regain its previous state. 

import { applyMiddleware, createStore } from "redux"; 

import { composeWithDevTools } from "redux-devtools-extension"; 

import createSagaMiddleware from "redux-saga"; 


import rootReducer from "./rootReducer"; 

import { configureSaga } from "./rootSaga"; 


import logger from "./loggerMiddleware"; 


function getPreviousState() { 

  const store = localStorage.getItem("store"); 
  
  if (store) { 
  
    try { 
    
      return JSON.parse(store); 
      
    } catch (e) { 
    
      return undefined; 
      
    } 
    
  } 
  
} 


export default (preloadedState = getPreviousState()) => { 

  const sagaMiddleware = createSagaMiddleware(); 
  
  const middlewares = [logger<any>(), sagaMiddleware]; 
  
  const middlewareEnhancer = applyMiddleware(...middlewares); 
  

  const composedEnhancers = composeWithDevTools({ serialize: true })( 
  
    middlewareEnhancer 
    
  ); 
  

  const store = createStore(rootReducer, preloadedState, composedEnhancers); 
  

  sagaMiddleware.run(configureSaga()); 
  
  window.onbeforeunload = function() { 
  
    localStorage.setItem("store", JSON.stringify(store.getState())); 
    
  }; 
  
  return store; 
  
}; 

