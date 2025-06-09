# unknow_stg

\= SPEC-001: Offline Interactive Story Game (Flutter)
\:sectnums:
\:toc:

\== Background

The need for immersive, offline, single-player games is rising, especially among mobile users seeking rich storytelling without requiring constant internet access. This project is a Flutter-based interactive storytelling game where players engage with a narrative that evolves based on their choices. At each step of the story, the player is presented with a set of decisions, and the chosen path influences the direction, tone, and conclusion of the story.

Inspired by successful apps like "Choices" and "Lifeline", this application is designed to run entirely offline and be fully self-contained, making it ideal for low-connectivity environments or on-the-go usage. The primary goal is to offer compelling storytelling with branching decision trees, allowing for replayability and personalized narratives.

\== Requirements

* MUST allow users to play a story-based game with multiple-choice decisions.
* MUST store and load stories locally (offline capability).
* MUST allow story branching based on player decisions.
* MUST persist player progress locally.
* SHOULD support visual elements (e.g., character images, background scenes).
* SHOULD allow saving and loading multiple storylines.
* COULD include audio narration or sound effects.
* COULD support downloadable story packs (for future extensibility).
* WON'T support multiplayer or real-time online features.

\== Method

The game architecture is centered around a decision-tree structure using story nodes. Each story is modeled as a JSON file and parsed into Dart objects.

\=== Data Model

Each story is composed of interconnected nodes:

## \[source,json]

{
"id": "start",
"text": "You wake up in a forest with no memory. What do you do?",
"choices": \[
{
"text": "Explore the forest",
"next": "explore\_forest"
},
{
"text": "Call for help",
"next": "call\_help"
}
]
}
-

## \[source,dart]

class StoryNode {
final String id;
final String text;
final List<StoryChoice> choices;

StoryNode({required this.id, required this.text, required this.choices});

factory StoryNode.fromJson(Map\<String, dynamic> json) => ...;
}

class StoryChoice {
final String text;
final String next;

StoryChoice({required this.text, required this.next});

factory StoryChoice.fromJson(Map\<String, dynamic> json) => ...;
}
-

\=== Architecture Components

## \[plantuml, story\_architecture, png]

@startuml
actor Player
rectangle "Flutter App" {
component "Story Engine" as Engine
component "UI Screens"
component "Local Storage (Hive or SQLite)"
database "Assets (images/audio)" as Assets
}

Player --> "UI Screens"
"UI Screens" --> Engine
Engine --> "Local Storage (progress + stories)"
Engine --> Assets
@enduml
-------

Updated Directory Structure with Feature-based Clean Architecture:

## \[source]

lib/
└── features/
└── story/
├── data/
│   └── story\_repository.dart
├── domain/
│   ├── story\_node.dart
│   ├── story\_choice.dart
│   ├── player\_progress.dart
│   └── usecases/
│       └── load\_story.dart (future)
├── presentation/
│   ├── bloc/
│   │   ├── story\_bloc.dart
│   │   ├── story\_event.dart
│   │   └── story\_state.dart
│   ├── screens/
│   │   ├── story\_screen.dart
│   │   └── story\_selection\_screen.dart
│   └── widgets/
│       └── choice\_button.dart (optional)
------------------------------------------

\=== Story Progression & Persistence

* On app start, the engine checks for existing saved progress for each story.
* If progress exists, the player can continue; otherwise, the story starts at the first node.
* Every time a choice is made, the following are saved:

  * `currentNodeId`
  * `pathTaken[]`
  * `storyId`

## \[source,dart]

class PlayerProgress {
final String storyId;
final String currentNodeId;
final List<String> pathTaken;
}
-

Example Hive storage:

* `box('progress')`:

  * key: `story_001`
  * value: `{"currentNodeId": "explore_forest", "pathTaken": ["start", "explore_forest"]}`

\=== Conditional Save Slots

At key nodes, story JSON can define `"checkpoint": true`. When reached:

## \[source,json]

{
"id": "betrayal\_choice",
"text": "Your friend betrays you. What do you do?",
"checkpoint": true,
"choices": \[
{"text": "Forgive", "next": "forgive\_path"},
{"text": "Leave them", "next": "leave\_path"}
]
}
-

## \[source,dart]

class Checkpoint {
final String id;
final String storyId;
final String nodeId;
final DateTime savedAt;
}
-

\=== UI Flow

\==== 1. Story Selection Screen

* Lists all available stories
* Displays title, summary, progress

\==== 2. Story Screen

* Shows background, story text, and choices
* Navigates to next node on selection

\==== 3. Progress Menu

* Restart story
* Resume from checkpoint
* View previous paths

## \[plantuml, story\_ui\_flow, png]

@startuml
start
\:Open App;
\:Load local stories;
if (Progress exists?) then (yes)
\:Show Continue button;
endif
\:Story Selection Screen;
\:User picks a story;
\:Load JSON file;
\:Start at saved node or beginning;
\:Render Story Screen;
while (Not at end)
\:User makes a choice;
\:Move to next node;
\:Save progress;
endwhile
\:Show Ending Screen;
stop
@enduml
-------

\== Implementation

\=== 1. Story Data Format

* Define JSON schema for story nodes
* Create 1–2 sample stories

\=== 2. Data Layer

* Setup Hive for local storage
* Create models and repository for stories and progress

\=== 3. Story Engine

* Load nodes and manage state transitions
* Save and load progress and checkpoints

\=== 4. UI Screens

* StorySelectionScreen
* StoryScreen
* ProgressMenu

\=== 5. Assets

* Add sample images and audio
* Load via pubspec.yaml

\=== 6. Testing

* Unit test parsing, navigation, saving
* Simulate multiple paths

\== Milestones

\=== Milestone 1: Core Engine MVP (Week 1–2)

* JSON parsing
* Story traversal
* Basic UI

\=== Milestone 2: Persistence Layer (Week 2–3)

* Hive integration
* Progress saving/loading

\=== Milestone 3: UI/UX Polish (Week 3–4)

* Story selection
* Visual assets

\=== Milestone 4: First Full Story + Test Coverage (Week 4–5)

* Complete story scenario
* Full path testing

\=== Milestone 5: Post-MVP Enhancements (Week 6+)

* Downloadable stories
* Audio, animations

\== Gathering Results

\=== Functional Validation

* All story paths work without errors
* Progress and checkpoints persist correctly

\=== User Testing Feedback

* Users complete story and evaluate UX

\=== Offline Capability Check

* No internet required for full playthrough

\=== Replayability Test

* Different choices yield distinct endings

\=== Performance

* App launch < 2s
* Node navigation < 100ms

