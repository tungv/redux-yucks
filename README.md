# redux-yucks
declarative redux ducks with FP

# Proposal

Given a yaml file like this:

```yaml
module: myAwesomeModule
imports:
  ./some/other/file: ACTIONS

states:
  currentStep: initial

selectors:
  isAtTheInitialStepSelector: currentStepSelector | eq('initial')

actionCreators:
  gotoInitialStep: constant('initial') | currentStepSetter
  gotoAwesomeStep: constant('awesome') | currentStepSetter

epics:
  updateStepWhenUpdateComplete:
    - filter  get('type') | eq(ACTIONS.AWESOME)
    - filter  get('payload.awesomeness') | gte(9000)
    - mapTo   currentStepSetter('success')
```

`redux-yucks` will take it as an input an returns the follow file

```js
import { combineReducers } from 'redux';
import { createAction, handleActions } from 'redux-actions';
import { ACTIONS } from './some/other/file';

import flow from 'lodash/fp/flow';
import get from 'lodash/fp/get';
import constant from 'lodash/fp/constant';
import eq from 'lodash/fp/eq';
import gte from 'lodash/fp/gte';
import 'rxjs/add/operator/filter';
import 'rxjs/add/operator/mapTo';

// currentStep
const takePayload = (state, { payload }) => payload;
const ACTION_SETTER_currentStep = 'myAwesomeModule/action/SETTER_currentStep';
export const currentStepSetter = createAction(ACTION_SETTER_currentStep);
const currentStep = handleActions({
  [ACTION_SETTER_currentStep]: takePayload
}, 'initial');

// selectors
export const isAtTheInitialStepSelector = flow(currentStepSetter, eq('initial'));

// actionCreators
export const gotoInitialStep = flow(constant('initial'), currentStepSetter);
export const gotoAwesomeStep = flow(constant('awesome'), currentStepSetter);

// combined reducer
export const reducer = combineReducers({
  currentStep,
});

// epics
const updateStepWhenUpdateComplete = actions$ =>
  actions$
    .filter(flow(get('type'), eq(ACTIONS.AWESOME)))
    .filter(flow(get('payload.awesomeness'), gte(9000)))
    .mapTo(currentStepSetter('success'));

export default { myAwesomeModule: reducer };
```
