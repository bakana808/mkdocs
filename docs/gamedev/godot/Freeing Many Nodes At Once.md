If you run into the problem of your game crashing when freeing a large amount (100s) of nodes at once, that is, trying to run something like:

```gdscript
for node in get_tree().get_nodes_in_group("group"):
	node.queue_free()
```

here is a workaround that might work.

## Batched `queue_free()`

The idea here is instead of freeing all of the nodes at once, only free some of them per tick:

```gdscript
const MAX_NODE_DELETIONS = 5
var nodes_queued_for_deletion := []

func free_nodes():
    # add all nodes you want to delete here
    for node in get_tree().get_nodes_in_group("group"):
        nodes_queued_for_deletion.append(node)

func _physics_process(_delta):
	# free batches of nodes at a time here
    for node in nodes_queued_for_deletion.slice(0, MAX_NODE_DELETIONS):
        if is_instance_valid(node): bullet.queue_free()
    nodes_queued_for_deletion = nodes_queued_for_deletion.slice(MAX_NODE_DELETIONS)
```