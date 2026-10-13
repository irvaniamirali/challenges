# Contributing

Thanks for your interest in contributing to this project!

This project aims to make learning DevOps, Linux, networking, and related technologies practical, approachable, and technically accurate.

Contributions of all sizes are welcome, including improving explanations, fixing errors, adding labs, and reviewing existing content.

## Ways to Contribute

You can contribute by:

- Adding a new lab or improving an existing one.
- Correcting technical mistakes or outdated instructions.
- Making explanations clearer and easier to understand.
- Improving commands, expected outputs, or troubleshooting steps.
- Reporting bugs or suggesting new topics.

## Guidelines

Please keep the following principles in mind when contributing:

- **Teach, don't just instruct.** Explain why commands are used and what their important options mean.
- **Be technically accurate.** Verify commands, explanations, and expected results before submitting a contribution.
- **Keep it practical.** Prefer hands-on experiments and realistic examples over purely theoretical explanations.
- **Explain the reasoning.** When a lab involves troubleshooting, help learners understand how to investigate the problem rather than simply giving them the fix.
- **Keep prerequisites reasonable.** Avoid unnecessary infrastructure, paid services, or complicated setup steps.
- **Keep it focused.** Each lab should have clear learning objectives and a manageable scope.
- **Be honest about limitations.** Document relevant assumptions, environment requirements, and differences in expected behavior.

## Lab Structure

Labs should be organized by topic under `labs/`.

For example:

```text
labs/
└── networking/
    └── 01-example-topic/
        └── README.md
```

Use descriptive directory names and keep each lab's content together.

Each lab should include relevant metadata, such as its level, category, estimated duration, and environment requirements, along with learning objectives, instructions, and any necessary cleanup steps.

The exact structure may vary depending on the lab's purpose.

## Commands and Technical Explanations

Commands should be presented in a way that learners can understand and reproduce.

- Explain non-obvious command options and flags.
- Include expected behavior or output when it helps learners verify their results.
- State which operating system or environment the instructions target.
- Avoid assuming that a command succeeded without providing a way to verify it.
- Never include real credentials, tokens, private keys, or other sensitive information.

Unless a lab explicitly states otherwise, its instructions should target Linux.

## Submitting Changes

1. Fork the repository.
2. Create a branch for your changes.
3. Make and verify your changes.
4. Open a pull request describing what you changed and why.

For new labs, include the problem or concept being taught, the intended learning outcomes, and any relevant environment requirements.

For fixes, explain what was incorrect and how your changes address it.

## Questions and Suggestions

If you have an idea for a new lab, notice a technical issue, or have suggestions for improving the project, open an issue to discuss it.

Not every contribution needs to be a large addition. Small, thoughtful improvements matter too.

Thanks for helping make technical learning more practical and accessible!

