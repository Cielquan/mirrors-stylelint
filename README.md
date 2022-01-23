# stylelint mirror


Mirror of stylelint package for pre-commit.

For pre-commit: see https://github.com/pre-commit/pre-commit

For stylelint: see https://github.com/stylelint/stylelint


### Using stylelint with pre-commit

Add this to your `.pre-commit-config.yaml`:

    -   repo: https://github.com/cielquan/stylelint
        rev: ''  # Use the sha / tag you want to point at
        hooks:
        -   id: stylelint

