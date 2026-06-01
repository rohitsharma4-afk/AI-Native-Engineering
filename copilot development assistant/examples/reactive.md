# Reactive Agent Example

## Objective

Demonstrate quick bug fixing and code improvement.

## Prompt

Use the Reactive Agent pattern.

Fix the following Spring Boot controller.

Requirements:

1. Add exception handling
2. Add logging
3. Return ResponseEntity
4. Follow Spring Boot best practices

@GetMapping("/users")
public List<User> getUsers() {
    return userRepository.findAll();
}


## Expected Outcome

* Missing exception handling detected
* Missing logging detected
* ResponseEntity added
* Improved controller generated

## Pattern Characteristics

* Fast response
* Minimal planning
* Direct action
