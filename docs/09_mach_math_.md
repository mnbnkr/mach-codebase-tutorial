---
layout: default
title: "Chapter 9: Math Library"
nav_order: 10
---

# Chapter 9: Math Library

> (`mach.math`)

> ***CAUTION**:* *THIS TUTORIAL WAS AI-GENERATED AND MAY CONTAIN ERRORS. IT IS **NOT** AFFILIATED WITH OR ENDORSED BY HEXOPS/MACH.*

Welcome to Chapter 9! In [Chapter 8: Audio Abstraction](08_mach_sysaudio_.md), we learned how Mach handles sound across different platforms. Now, let's shift gears to something fundamental to almost all game and graphics programming: mathematics!

## Your Specialized Geometry Toolkit

Imagine you're building a 2D game where a character needs to move around the screen, or a 3D game where objects rotate and scale. How do you calculate their positions, orientations, and sizes? How do you represent colors or directions?

You need tools to work with numbers in the context of 2D and 3D space. That's precisely what `mach.math` provides. Think of it as a specialized **calculator and ruler set** designed specifically for the geometry needed in game development.

It offers essential mathematical building blocks:

*   **Vectors:** Like arrows pointing in space, used for positions, directions, velocities, or even colors.
*   **Matrices:** Grids of numbers used primarily for transforming things (moving, rotating, scaling).
*   **Quaternions:** A special way to represent rotations in 3D space.

`mach.math` isn't just *any* math library; it has specific, opinionated rules (**conventions**) about how it represents things like coordinate systems and how data is stored. This ensures that all calculations done using `mach.math`, both within the engine itself and in your application code, are consistent and predictable.

## Key Concepts: The Building Blocks

Let's explore the main tools in the `mach.math` toolkit:

*   **Vectors (`Vec2`, `Vec3`, `Vec4`)**
    *   Think of a vector as an arrow with a direction and a length.
    *   `Vec2`: A 2D vector (x, y). Perfect for representing positions or sizes on a 2D screen.
    *   `Vec3`: A 3D vector (x, y, z). Used for positions, directions, or scaling factors in 3D space.
    *   `Vec4`: A 4D vector (x, y, z, w). Often used for colors (Red, Green, Blue, Alpha), or for advanced 3D calculations involving homogeneous coordinates (don't worry too much about that for now!).

*   **Matrices (`Mat3x3`, `Mat4x4`)**
    *   Think of a matrix as a grid of numbers arranged in rows and columns.
    *   Their superpower is *transforming* vectors. Multiplying a vector by a matrix can move (translate), rotate, or scale the vector's position or direction.
    *   `Mat3x3`: A 3x3 matrix. Useful for 2D transformations or manipulating texture coordinates (UVs).
    *   `Mat4x4`: A 4x4 matrix. The workhorse for 3D transformations. It can combine translation, rotation, and scaling into a single entity.

*   **Quaternions (`Quat`)**
    *   These are a more advanced way to represent 3D rotations (an alternative to using matrices for rotation).
    *   They are often preferred for smoothly interpolating between rotations and avoiding a problem called "gimbal lock" (where you can lose a degree of rotational freedom). They are represented by 4 numbers (x, y, z, w).

*   **Important Conventions:** `mach.math` makes specific choices to ensure consistency:
    *   **Coordinate System (+Y Up, Left-Handed):** This defines the direction of axes in 3D space. Imagine holding your **left hand** in a thumbs-up:
        *   Thumb points **UP** (+Y)
        *   Fingers curl towards **FORWARD** (+Z)
        *   Your palm faces **LEFT** (+X)
        *   This system is used consistently for world space coordinates. You can read more details and see diagrams here: [Coordinate system](../math/coordinate-system.md).
    *   **Matrix Storage (Column-Major):** This defines how the numbers in a matrix are laid out in computer memory. Mach stores matrices *column by column*.
    *   **Vector Type (Column Vectors):** This affects how matrix multiplication works. Mach uses `Matrix * Vector = TransformedVector`.
    *   These choices match the conventions used by modern graphics APIs like WebGPU and Vulkan, and libraries like OpenGL. It differs from the conventions used by DirectX and Unreal Engine. For a deeper dive, see [Matrix storage](../math/matrix-storage.md).

## Putting `mach.math` to Work

Let's see how to use these tools for basic tasks.

**1. Using the Library**

First, you typically import Mach and create a convenient alias for the math module:

```zig
// src/App.zig (or any .zig file)
const mach = @import("mach");
const math = mach.math; // Alias for easier use
```

**2. Creating and Using Vectors**

Vectors are straightforward to create and manipulate.

```zig
// Create a 2D position vector
const player_pos = math.vec2(100.0, 50.0); // x = 100.0, y = 50.0

// Create a 3D direction vector
const move_dir = math.vec3(0.0, 0.0, 1.0); // Pointing forward along +Z

// Create a color vector (Red, Green, Blue, Alpha)
const red_color = math.vec4(1.0, 0.0, 0.0, 1.0); // Fully red, fully opaque

std.log.info("Player starts at: {any}", .{player_pos});
```

*   `math.vec2()`, `math.vec3()`, `math.vec4()` are functions to initialize vectors with specific values.

You can perform operations on vectors:

```zig
// Calculate a new position by adding a movement vector
const movement = math.vec2(5.0, 0.0); // Move 5 units right
const new_pos = player_pos.add(&movement); // new_pos = (105.0, 50.0)

// Scale the movement vector by time (e.g., frame delta time)
const dt: f32 = 0.016;
const scaled_movement = movement.mulScalar(dt); // Small movement amount

// Calculate the length (magnitude) of a vector
const dir_len = move_dir.len(); // Length is 1.0 for a normalized direction

std.log.info("New position: {any}", .{new_pos});
std.log.info("Movement scaled by time: {any}", .{scaled_movement});
std.log.info("Direction length: {any}", .{dir_len});
```

*   `.add()`, `.sub()`, `.mul()`, `.div()` perform element-wise operations between two vectors.
*   `.mulScalar()`, `.divScalar()` multiply or divide all elements by a single number (scalar).
*   `.len()` calculates the geometric length of the vector.

**3. Creating and Using Matrices**

Matrices are often used for transformations.

```zig
// Create an identity matrix (represents "no transformation")
const identity_mat = math.Mat4x4.ident;

// Create a translation matrix (moves things)
const translate_mat = math.Mat4x4.translate(math.vec3(10.0, 5.0, 0.0));

// Create a scaling matrix (resizes things)
const scale_mat = math.Mat4x4.scale(math.vec3(2.0, 2.0, 2.0)); // Double size

std.log.info("Identity Matrix:\n{any}", .{identity_mat});
std.log.info("Translate Matrix:\n{any}", .{translate_mat});
```

*   `Mat4x4.ident` gives you a matrix that doesn't change anything it multiplies.
*   `Mat4x4.translate()` creates a matrix that will shift points by the given vector.
*   `Mat4x4.scale()` creates a matrix that will scale points by the given factors.

**4. Combining Transformations**

You can combine transformations by multiplying matrices. Remember: **Order matters!** Multiplying `A * B` is generally *not* the same as `B * A`. In Mach (using column vectors), transformations apply right-to-left. If you want to *first* scale an object and *then* translate it, the multiplication order is `Translate * Scale`.

```zig
// Combined: first scale, then translate
const scale_then_translate = translate_mat.mul(&scale_mat);

// Combined: first translate, then scale
const translate_then_scale = scale_mat.mul(&translate_mat);

std.log.info("Scale then Translate:\n{any}", .{scale_then_translate});
std.log.info("Translate then Scale:\n{any}", .{translate_then_scale});
// Note: The resulting numbers in the matrices will be different!
```

*   `matrixA.mul(&matrixB)` multiplies matrix A by matrix B.

**5. Transforming Vectors with Matrices**

The most common use of matrices is to transform vectors (like vertex positions).

```zig
// A point in 3D space (e.g., a corner of a cube)
const point = math.vec4(1.0, 1.0, 0.0, 1.0); // Using Vec4 for 4x4 matrix math

// Transform the point using the combined matrix
const transformed_point = scale_then_translate.mulVec(&point);

std.log.info("Original point: {any}", .{point});
// Expected output: (scaled x=2, y=2) then (translated x+10, y+5) -> (12.0, 7.0, 0.0, 1.0)
std.log.info("Transformed point: {any}", .{transformed_point});
```

*   `matrix.mulVec(&vector)` multiplies the matrix by the vector, applying the matrix's transformation to the vector. The `w` component (the last number in `Vec4`) is important for translations in 4x4 matrices; it's usually 1.0 for positions and 0.0 for directions.

**6. Quaternions (Briefly)**

Quaternions are mainly for rotation.

```zig
// Identity quaternion (no rotation)
const no_rotation = math.Quat.identity();

// Create a quaternion representing a 90-degree rotation around the Y axis
const ninety_deg_y_rot = math.Quat.fromAxisAngle(
    math.vec3(0.0, 1.0, 0.0), // Y axis
    math.degreesToRadians(90.0), // Angle
);

// You can combine rotations by multiplying quaternions
const combined_rot = ninety_deg_y_rot.mul(&no_rotation); // Still 90 deg Y rot

std.log.info("90 deg Y rotation quat: {any}", .{ninety_deg_y_rot});
```

*   `Quat.identity()` gives no rotation.
*   `Quat.fromAxisAngle()` creates a rotation around a specified axis.
*   You can convert quaternions to rotation matrices (`Mat4x4.rotateByQuaternion`) when needed to combine them with translation and scaling.

## Under the Hood: Structure and Performance

How is `mach.math` built?

**Implementation Overview:**

*   **Built on Zig Vectors:** `mach.math` leverages Zig's built-in, highly optimized vector types (`@Vector(N, T)`). This means operations like vector addition (`a.v + b.v`) often compile down to efficient SIMD instructions (Single Instruction, Multiple Data), allowing the CPU to perform calculations on multiple components (x, y, z, w) simultaneously.
*   **Struct Wrappers:** The types like `Vec3` and `Mat4x4` are `extern struct` wrappers around these built-in vectors or arrays of vectors. This provides a clean API with named functions (`.add()`, `.mulScalar()`, `.translate()`) while keeping the data layout efficient and predictable.
*   **Column-Major Layout:** `Mat4x4` is essentially an array of four `Vec4`s, where each `Vec4` represents a *column* of the matrix.

    ```
    Mat4x4.v: [4]Vec4 = [
        Column0 (Vec4),
        Column1 (Vec4),
        Column2 (Vec4),
        Column3 (Vec4), // Translation often lives here (tx, ty, tz, tw)
    ]
    ```
    When you use `Mat4x4.init(...)`, it takes *rows* as input for easier visualization but stores them internally in this column-major format.
*   **Convention Enforcement:** The functions provided (`.translate()`, `.scale()`, `.projection2D()`, etc.) are designed to work correctly with the engine's chosen left-handed, +Y up coordinate system and column-vector convention.

**Code Glance:**

Let's peek at how some types and functions might be structured:

*   **`src/math/main.zig`:** Defines the public types and aliases.

    ```zig
    // src/math/main.zig (Simplified Snippet)
    const vec = @import("vec.zig");
    const mat = @import("mat.zig");
    const q = @import("quat.zig");

    // Standard f32 precision types
    pub const Vec2 = vec.Vec2(f32);
    pub const Vec3 = vec.Vec3(f32);
    pub const Vec4 = vec.Vec4(f32);
    pub const Quat = q.Quat(f32);
    pub const Mat3x3 = mat.Mat3x3(f32);
    pub const Mat4x4 = mat.Mat4x4(f32);

    // Standard f32 precision initializers (convenience functions)
    pub const vec2 = Vec2.init;
    pub const vec3 = Vec3.init;
    pub const vec4 = Vec4.init;
    pub const quat = Quat.init;
    pub const mat3x3 = Mat3x3.init;
    pub const mat4x4 = Mat4x4.init;
    ```
    *   This file brings together the definitions from `vec.zig`, `mat.zig`, etc., and provides easy-to-use initializers like `math.vec3(...)`.

*   **`src/math/vec.zig`:** Defines vector types and operations.

    ```zig
    // src/math/vec.zig (Simplified Snippet for Vec3)
    pub fn Vec3(comptime Scalar: type) type {
        return extern struct {
            v: @Vector(3, Scalar), // The core data using Zig's built-in vector

            pub const n = 3;
            pub const T = Scalar;
            const VecN = @This();

            // Initializer function
            pub inline fn init(xs: Scalar, ys: Scalar, zs: Scalar) VecN {
                return .{ .v = .{ xs, ys, zs } };
            }

            // Accessor functions
            pub inline fn x(v: *const VecN) Scalar { return v.v[0]; }
            pub inline fn y(v: *const VecN) Scalar { return v.v[1]; }
            pub inline fn z(v: *const VecN) Scalar { return v.v[2]; }

            // Example operation using built-in vector math
            pub inline fn add(a: *const VecN, b: *const VecN) VecN {
                return .{ .v = a.v + b.v }; // Zig handles element-wise addition
            }
            // ... other functions like sub, mulScalar, len, normalize ...
        };
    }
    ```
    *   Shows how `Vec3` wraps `@Vector(3, f32)` and how basic operations like `add` directly use Zig's efficient vector arithmetic.

*   **`src/math/mat.zig`:** Defines matrix types and operations.

    ```zig
    // src/math/mat.zig (Simplified Snippet for Mat4x4)
    pub fn Mat4x4(comptime Scalar: type) type {
        return extern struct {
            // Stores 4 columns, each being a Vec4
            v: [4]vec.Vec4(Scalar),

            pub const cols = 4;
            pub const rows = 4;
            pub const T = Scalar;
            pub const Vec = vec.Vec4(Scalar);
            // ... other constants ...
            const Matrix = @This();

            // Identity matrix definition
            pub const ident = Matrix.init( // Calls init below
                &Vec.init(1, 0, 0, 0),
                &Vec.init(0, 1, 0, 0),
                &Vec.init(0, 0, 1, 0),
                &Vec.init(0, 0, 0, 1),
            );

            // init takes rows for user convenience...
            pub inline fn init(r0: *const Vec, r1: *const Vec, r2: *const Vec, r3: *const Vec) Matrix {
                // ...but stores them transposed into columns!
                return .{ .v = [_]Vec{
                    Vec.init(r0.x(), r1.x(), r2.x(), r3.x()), // Column 0
                    Vec.init(r0.y(), r1.y(), r2.y(), r3.y()), // Column 1
                    Vec.init(r0.z(), r1.z(), r2.z(), r3.z()), // Column 2
                    Vec.init(r0.w(), r1.w(), r2.w(), r3.w()), // Column 3
                } };
            }

            // Creates a translation matrix directly in column-major format
            pub inline fn translate(t: vec.Vec3(T)) Matrix {
                return init( // Uses the init function above
                    &Vec.init(1, 0, 0, t.x()), // Row 0 -> affects Col 3
                    &Vec.init(0, 1, 0, t.y()), // Row 1 -> affects Col 3
                    &Vec.init(0, 0, 1, t.z()), // Row 2 -> affects Col 3
                    &Vec.init(0, 0, 0, 1),     // Row 3 -> affects Col 3
                );
            }

            // Matrix multiplication (simplified concept)
            pub inline fn mul(a: *const Matrix, b: *const Matrix) Matrix {
                 var result: Matrix = undefined;
                 // Perform row-by-column dot products to calculate result cells
                 // Store results correctly in the column-major 'result.v'
                 // ... multiplication logic ...
                 return result;
            }

            // Matrix-vector multiplication (simplified concept)
            pub inline fn mulVec(matrix: *const Matrix, vector: *const Vec) Vec {
                 var result = Vec.splat(0);
                 // Perform dot products of matrix rows with the vector
                 // ... multiplication logic ...
                 return result;
            }
            // ... other functions like scale, rotate, projection2D ...
        };
    }
    ```
    *   Highlights the column-major storage (`v: [4]Vec`) and how `init` transposes input rows into stored columns.
    *   Shows how functions like `translate` construct the matrix according to mathematical conventions, which `init` then correctly stores column-wise.

## Conclusion

You've learned about `mach.math`, the engine's dedicated library for handling the essential mathematics of 2D and 3D geometry. It provides core types like vectors (`Vec2`, `Vec3`, `Vec4`), matrices (`Mat3x3`, `Mat4x4`), and quaternions (`Quat`), along with functions for common operations like translation, rotation, scaling, and vector math. Critically, it establishes consistent conventions (+Y up, left-handed, column-major matrices) used throughout the engine, making calculations predictable. This library is the foundation for positioning, orienting, and animating objects in your Mach applications.

We've covered many of the core runtime concepts of Mach. Now, how do we actually *build* our Mach application? How do we tell Zig how to compile our code, include assets, and link against Mach itself?

Let's move on to [Chapter 10: Build System](10_build_system_.md).

---

Generated by [AI Codebase Knowledge Builder](https://github.com/mnbnkr/Tutorial-Codebase-Knowledge)
