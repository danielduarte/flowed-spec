# Flowed Spec

**Version 1.0.0-rc.1**

## Introduction

This document defines the standard specification for flow definitions in [Flowed] format, shortly named "Flowed documents".

This specification (from now on called "Flowed Spec") is versioned using [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html).


## Definitions

List of relevant definitions of terms and phrases used in this document applicable in this context.

**Flowed**: A configurable task executor engine to manage dependencies automatically, maximizing parallelism. Task are ran by providing a *Flowed Document* to the engine.

**Task**: Minimal unit of execution for [Flowed] flows.

**Flow**: A flow definition represented by a document that follows this specification.

**Flowed Document**: A JSON document that follows this specification.


## Specification

### Versioning

All Flowed documents with no version specification are by default in version 1.0.0.
Future versions of this specification will have the possibility to specify the version in Flowed documents.


### Format

The Flowed documents are [JSON](https://www.json.org/json-en.html) documents that conform the schema defined in the section "Schema" of this document.

All the `string` values and object keys in a Flowed document are case-sensitive.


### Schema

#### FlowedSpec

Root object of the Flowed document.

:warning: define Map, Array, string

Fields

| Name      | Type                      | Required | Description                                                                                                                                                                                                                                                                                                                    |
|-----------|---------------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `tasks`   | `Map<TaskCode, TaskSpec>` | No       | A map with all the tasks definitions of the flow, following the specification of `TaskSpec`. The keys of this map are the unique identifiers of each task definition, in this context called "task codes". These keys follow the specification of `TaskCode`. The order of the tasks definitions in this object is irrelevant. |
| `options` | `FlowedOptions`           | No       | Object to specify running options for this flow definition.                                                                                                                                                                                                                                                                    |


#### TaskSpec

Task definition specification.
A task definition object represents a unit of execution in the flow context that can be run 0, 1 or more times, depending on the flow definition, the execution parameters, and each particular execution evolution.
The predictability of execution (or non-execution) of each task depends on how the entire flow is defined.

| Name            | Type                      | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
|-----------------|---------------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `requires`      | `Array<ValueName>`        | No       | List of value names required for the task execution. These value names follow the specification of `ValueName`. The order of the value names is irrelevant.                                                                                                                                                                                                                                                                                                                          |
| `provides`      | `Array<ValueName>`        | No       | List of value names expected to be provided by the task execution on its termination. These value names follow the specification of `ValueName`. The order of the value names is irrelevant. Task executions are not required to provide all the specified values in this list, but additional values cannot be provided. If a task implementation returns additional unknown values, they are ignored outside of the scope of the task, without altering the execution of the flow. |
| `defaultResult` | `AnyValue`                | No       | Value to be returned for all non-provided value names in the `provides` list. If `defaultResult` is not specified, the non-provided values are not supplied. If the value given in `defaultResult` is a pointer (for example, a JavaScript object reference), the same pointer is returned for all non-provided values. In other words, no copies of the default result value are made.                                                                                              |
| `resolver`      | `TaskResolverSpec`        | No       | Specification of the resolver for the task. Holds the required information to find the code unit that executes (resolves) the task, and map the input parameters and output results.                                                                                                                                                                                                                                                                                                 |



#### TaskCode

Values used as identifiers for the tasks in the flow scope. These identifiers are called "task codes".
The tasks codes follows the specification of `Identifier`s.


#### ValueName

Identifiers of the values or tokens going through the flow as input parameters and/or output results.
The value names follows the specification of `Identifier`s.


#### Identifier

`string` values used as identifiers in different parts of a Flowed document.
The identifiers in [Flowed] must follow these rules:
- Length: from 1 to 64 inclusive.
- Valid characters: Letters in `[a-z, A-Z]`, digits (`[0-9]`), underscore (`_`) and hyphen (`-`).


#### FlowedOptions

Object with options to configure parameters of execution for the flow.
They are not intended to be modified between different executions, since they are considered part of the flow specification.
Implementations of this specification like [the official](https://github.com/danielduarte/flowed) one, can support additional execution parameters to control details of individual executions for the same flow.

| Name                           | Type      | Required | Description                                                                                                                                                                                                 |
|--------------------------------|-----------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `throwErrorOnUnsolvableResult` | `boolean` | No       | Indicates if an error must be thrown when a task does not provide some of the results declared in its `provides` list. When such errors are thrown, the flow execution is interrupted. Defaults to `false`. |
| `resolverAutomapParams`        | `boolean` | No       | Indicates if input parameters passed to the resolver should be mapped automatically to the value names in the `requires` list. Defaults to `false`.                                                         |
| `resolverAutomapResults`       | `boolean` | No       | Indicates if input results given by the resolver should be mapped automatically to the value names in the `provides` list. Defaults to `false`.                                                             |

#### AnyValue

Literally any possible value valid in the ECMAScript specification (in the version where the flow engine is run) that can be referenced by a variable.
It should be taking into account that, even when any reference-able ECMAScript value can be used as a value in a flow execution, flow states will be able to be serializable only when all its values are serializable.


#### TaskResolverSpec

| Name      | Type             | Required | Description                                                                                                                                                    |
|-----------|------------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`    | `ResolverName`   | Yes      | The name that identifies the resolver class or function to execute (resolve) the task. The same resolver could be used in any number of tasks in the same flow |
| `params`  | `TaskParamsMap`  | No       | The map where task parameters are provided. It maps flow value, literal values or transformations of them into task parameters by name                         |
| `results` | `TaskResultsMap` | No       | The map where task results are retrieved. It maps task results into flow values by name                                                                        |


#### ResolverName

Identifiers of resolver classes or functions that are executed to run a task. All tasks in a flow must have a resolver identified by its name.
The resolver names could have two formats:
- **Simple**: Following the specification of `Identifier`. For example: `"CallApi"`, `"MergeResults"`, `"calculateAvg"`.
- **Namespaced**: Formed by an `Identifier`, followed by `"::"` and then another `Identifier`. No spaces are admitted between those parts. For example: `"flowed::Echo"`, `"flowed::Conditional"`, `"MyCustomLib::Any"`.


#### TaskParamsMap

A map with of the form `Map<Identifier, ParamValueSpec>`.
The `Identifier`s used as keys should match the keys of the object taken as parameter by the resolver.
The values in the map specify how the parameter values are generated to be passed to the resolver.

| Name                      | Type             | Required | Description                                                                                         |
|---------------------------|------------------|----------|-----------------------------------------------------------------------------------------------------|
| {paramName}: `Identifier` | `ParamValueSpec` | No       | The specifications of the value or transformation to be mapped to the "`paramName`" task parameter. |


#### ParamValueSpec

The specification of a flow value, literal value or transformation used to be passed to a task as parameter.
One of:
- `ValueName`: A reference to one of the values in the `requires` clause.
- `ParamValueSpecLiteral`: An object of the form `{ "value": <aValue> }`, where "<aValue>" is any value that follows the spec `AnyValue` and represents the literal value to be used as a task parameter.
- `ParamValueSpecTransform`: An object of the form `{ "transform": <aTransformation>}`, where "<aTransformation>" is any _Flowed ST_ transformation template that represents the transformation to be evaluated and then used as a task parameter.


#### TaskResultsMap

A map with of the form `Map<Identifier, AnyValue>`.
The `Identifier`s used as keys should match the keys of the object returned as result by the resolver.
The values in the map specify how the result values are passed to the flow as flow values.

| Name                      | Type         | Required | Description                                                                                        |
|---------------------------|--------------|----------|----------------------------------------------------------------------------------------------------|
| {paramName}: `Identifier` | `AnyValue`   | No       | The specifications of the value or transformation to be mapped to the "`paramName`" task parameter |


## License

This spec is versioned under the MIT license.
Note that this license applies to the Flowed Spec, which is not the same as the Node.js Flowed implementation.


## History

Flowed Spec versions and revisions. Note that this versions correspond to the Flowed Spec and not to the Flowed Node.js implementation.

| Version       | Change                   |
|---------------|--------------------------|
| `1.0.0-rc.1`  | `Created first revision` |
