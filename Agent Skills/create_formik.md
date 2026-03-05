---
name: Formik Form Component
description: Creates a standardized form component using Formik, Yup, and Ant Design.
---

# Formik Form Component Skill

This skill ensures all forms in the application follow a consistent validation and UI pattern.

## Usage

When the user asks to "create a form", "add validation", or "build an input form", follow these steps.

## Steps

1.  **Define Validation Schema**:
    - Import `* as Yup` from `yup`.
    - Create a schema matching the DTO: `const [Feature]Schema = Yup.object().shape({ ... })`.
2.  **Create Component**:
    - Export a functional component `[Feature]Form`.
    - Accept `initialValues` and `onSubmit` as props.
3.  **Implement Formik**:
    - Wrap the return JSX in `<Formik ...>`.
    - Pass `initialValues`, `validationSchema`, and `onSubmit`.
4.  **Build UI with Ant Design**:
    - Use `Form` from `antd` (or `formik-antd` if available, otherwise manual binding).
    - For each field, use a wrapper or manual `field`/`meta` binding.
    - **Important**: Use Name matching the schema keys.
    - Show validation errors using `meta.touched && meta.error`.
5.  **Submission Handling**:
    - Ensure the submit button has `type="submit"` and loading state `isSubmitting`.

## Example Code

```typescript
"use client";
import React from "react";
import { Formik, Form, Field, ErrorMessage } from "formik";
import * as Yup from "yup";
import { Button, Input } from "antd";

const SignupSchema = Yup.object().shape({
  email: Yup.string().email("Invalid email").required("Required"),
});

export const SignupForm = ({ onSubmit }) => (
  <Formik
    initialValues={{ email: "" }}
    validationSchema={SignupSchema}
    onSubmit={onSubmit}
  >
    {({ isSubmitting, handleChange, values, errors, touched }) => (
      <Form>
        <div className="mb-4">
          <Input
            name="email"
            value={values.email}
            onChange={handleChange}
            status={touched.email && errors.email ? "error" : ""}
          />
          <div className="text-red-500 text-sm">
            <ErrorMessage name="email" />
          </div>
        </div>
        <Button type="primary" htmlType="submit" loading={isSubmitting}>
          Submit
        </Button>
      </Form>
    )}
  </Formik>
);
```
