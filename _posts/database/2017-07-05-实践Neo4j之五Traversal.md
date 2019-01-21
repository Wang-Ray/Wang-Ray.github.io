---
layout: post
title: "实践Neo4j之五Traversal"
date: 2017-07-05 09:00:00 +0800
categories: 数据库
tags: database nosql neo4j
---

## GlobalGraphOperations

```java
try (Transaction tx = graphDb.beginTx()) {
    GlobalGraphOperations ggo = GlobalGraphOperations.at(graphDb);
    for (Relationship r : ggo.getAllRelationships()) {
    	r.delete();
	}
    for (Node n : ggo.getAllNodes()) {
    	n.delete();
    }
	tx.success();
}
```

## Traversal

### Using Neoj4 Core Java API

```java
Node node1 = graphDb.getNodeById(0)；
RelationShip rs1 = graphDb.getRelationShipById(1);
```

Node和RelationShip也自带一些一级检索方法，具体参见前文。

### Using Neo4j Traversal API

```java
TraversalDescription traversalMoviesFriendsLike = graphDb.traversalDescription()
				.relationships(MyRelationships.IS_FRIEND_OF).relationships(MyRelationships.HAS_SEEN, Direction.OUTGOING)
				.depthFirst().uniqueness(Uniqueness.NODE_GLOBAL).evaluator(Evaluators.atDepth(2))
				.evaluator(new Evaluator() {
					public Evaluation evaluate(Path path) {
						Node endNode = path.endNode();
						if (!endNode.hasProperty("type") || endNode.getProperty("type").equals("Movie")) {
							return Evaluation.EXCLUDE_AND_PRUNE;
						}
						for (Relationship r : endNode.getRelationships(Direction.INCOMING, MyRelationships.HAS_SEEN)) {
							if (r.getStartNode().equals(userJohn)) {
								return Evaluation.EXCLUDE_AND_CONTINUE;
							}
						}
						return Evaluation.INCLUDE_AND_CONTINUE;
					}
				});

Iterable<Node> moviesFriendsLike = traversalMoviesFriendsLike.traverse(userJohn).nodes();
```


