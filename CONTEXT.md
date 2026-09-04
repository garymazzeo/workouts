# Workout Recommendation

This context describes the workout catalog and the training history used to choose a suitable next session.

## Language

**Workout**:
A reusable exercise prescription in the catalog, such as “Barbell Cycling EMOM” or “Active Recovery Flow.”
_Avoid_: Training session, workout event

**Training Session**:
A dated occurrence in which a person completes or intends to complete a Workout.
_Avoid_: Workout, catalog entry

**Recommendation Context**:
Temporary user-provided information about recent or planned Training Sessions, used for one recommendation request and not retained by the product.
_Avoid_: Training history, workout history
