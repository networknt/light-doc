---
title: "Create Multi-Module Maven Project"
date: 2018-01-07T16:28:00-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Let's create a todo-list folder and copy the pom.xml from the existing project.
```
cd ~/networknt/light-example-4j/eventuate
mkdir todo-list
cd todo-list
cp ../todo-list.bak/pom.xml .

```

Now let's copy common, command and query modules to the todo-list folder from existing
one.

- common contains events and model
- command contains command definitions and command side service
- query contains query side service
 

```
cd ~/networknt/light-example-4j/eventuate/todo-list
cp -r ../todo-list.bak/common .
cp -r ../todo-list.bak/command .
cp -r ../todo-list.bak/query .

```

Now let's open pom.xml to remove other modules and only leave common, command and query:

```
    <modules>
        <module>common</module>
        <module>command</module>
        <module>query</module>
    </modules>

```

Let's run the maven build to make sure these three modules can be built.

```
cd ~/networknt/light-example-4j/eventuate/todo-list
mvn clean install
```

### Common module

common module defines domain object and event object across module both command side and query side

The top level event class define entity annotations:

```
@EventEntity(entity = "com.networknt.eventuate.todolist.domain.TodoAggregate")
public interface TodoEvent extends Event {
}
```

by default, light-eventuate-4j event sourcing framework will use the annotation defined 
"com.networknt.eventuate.todolist.domain.TodoAggregate" as entity type for entity table and 
topic name for Kafka.


### Command side API

Command side API implements aggregate to process command and apply events. For todolist sample, it 
simply return TodoInfo object:

```
public class TodoAggregate extends ReflectiveMutableCommandProcessingAggregate<TodoAggregate, TodoCommand> {

    private TodoInfo todo;

    private boolean deleted;

    public List<Event> process(CreateTodoCommand cmd) {
        if (this.deleted) {
            return Collections.emptyList();
        }
        return EventUtil.events(new TodoCreatedEvent(cmd.getTodo()));
    }

    public List<Event> process(UpdateTodoCommand cmd) {
        if (this.deleted) {
            return Collections.emptyList();
        }
        return EventUtil.events(new TodoUpdatedEvent(cmd.getTodo()));
    }

    public List<Event> process(DeleteTodoCommand cmd) {
        if (this.deleted) {
            return Collections.emptyList();
        }
        return EventUtil.events(new TodoDeletedEvent());
    }


    public void apply(TodoCreatedEvent event) {
        this.todo = event.getTodo();
    }

    public void apply(TodoUpdatedEvent event) {
        this.todo = event.getTodo();
    }

    public void apply(TodoDeletedEvent event) {
        this.deleted = true;
    }

    public TodoInfo getTodo() {
        return todo;
    }

}

```

### Query side API

Query side API implements event subscriber and process. It defines event handler for process events:

```
@EventSubscriber(id = "todoQuerySideEventHandlers")
public class TodoQueryWorkflow {

    private TodoQueryService service =
            (TodoQueryService)SingletonServiceFactory.getBean(TodoQueryService.class);

    public TodoQueryWorkflow() {
    }

    @EventHandlerMethod
    public void create(DispatchedEvent<TodoCreatedEvent> de) {
        TodoInfo todo = de.getEvent().getTodo();
        service.save(de.getEntityId(), todo);
    }

    @EventHandlerMethod
    public void delete(DispatchedEvent<TodoDeletedEvent> de) {
        service.remove(de.getEntityId());
    }

    @EventHandlerMethod
    public void update(DispatchedEvent<TodoUpdatedEvent> de) {
        TodoInfo todo = de.getEvent().getTodo();
        service.save(de.getEntityId(), todo);
    }
}
```

The framework will base on the event handler definition to decide which handler will be used 
to process the events.


