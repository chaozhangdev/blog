---
title: Research Topic - Computer Graphics
description: Ray tracing & volume rendering.
date: 2019-08-24
---

I would like to write this blog to thank [Dr. Xue Dong Yang](http://www2.cs.uregina.ca/~yang/) during my graduate studies in computer graphics.

He instructed us step by step to fully understand `Ray Tracing` & `Volumn Rendering` algorithms and write the source code to realize them.

<!-- more  -->

## Ray Tracing

### Main

```java
import java.io.*;
public class MainBody {
    public static void main(String[] args) throws IOException {
        int i, j;
        double C = 0;

        //get transform matrix
        MyModel.Mwc = Mwc_Mcw.Mwc(
          MyModel.VRP[0], MyModel.VRP[1], MyModel.VRP[2],
          MyModel.VPN[0], MyModel.VPN[1], MyModel.VPN[2],
          MyModel.VUP[0], MyModel.VUP[1], MyModel.VUP[2]);

        MyModel.Mcw = Mwc_Mcw.Mcw(
          MyModel.VRP[0], MyModel.VRP[1], MyModel.VRP[2],
          MyModel.VPN[0], MyModel.VPN[1], MyModel.VPN[2],
          MyModel.VUP[0], MyModel.VUP[1], MyModel.VUP[2]);

        //scan each pixel in the image
        for (i = 0; i < MyModel.ROWS; i++)
            for (j = 0; j < MyModel.COLS; j++) {
                // construct the ray, V, started from the CenterOfProjection, P0,
                // and passing through the pixel (i, j);
                ray_construction.generate(j, i);
                C = ray_tracing.main(MyModel.P0, MyModel.V0);
                if (C != 0)
                    MyModel.image[j][i] = (int) C;
            }
        // output the final image;
        FileOutputStream out = null;
        out = new FileOutputStream("C:/Users/CHAO/Desktop/output_image.raw");
        for (i = 0; i < MyModel.ROWS; i++)
            for (j = 0; j < MyModel.COLS; j++)
                out.write(MyModel.image[j][i]);
        out.close();
    }
}
```

### MWC -> MCW

```java
public class Mwc_Mcw {
    public static double[][] get_matrix(
      double VRP_x, double VRP_y, double VRP_z,
      double VPN_x, double VPN_y, double VPN_z,
      double VUP_x, double VUP_y, double VUP_z, String mark) {

        double u_x, u_y, u_z;
        double v_x, v_y, v_z;
        double n_x, n_y, n_z;
        double n_value, u_value, v_value;
        double R[][] = new double[4][4];
        double T[][] = new double[4][4];
        double M[][] = new double[4][4];

        // n = VPN / |VPN|
        n_value = Math.sqrt(VPN_x * VPN_x + VPN_y * VPN_y + VPN_z * VPN_z);
        n_x = VPN_x / n_value;
        n_y = VPN_y / n_value;
        n_z = VPN_z / n_value;

        // u = VUP X VPN / |VUP X VPN|
        u_value =
          Math.pow((VUP_y * VPN_z - VPN_y * VUP_z), 2) +
          Math.pow((VPN_x * VUP_z - VUP_x * VPN_z), 2) +
          Math.pow((VUP_x * VPN_y - VPN_x * VUP_y), 2);
        u_value = Math.sqrt(u_value);

        u_x = (VUP_y * VPN_z - VPN_y * VUP_z) / u_value;
        u_y = (VPN_x * VUP_z - VUP_x * VPN_z) / u_value;
        u_z = (VUP_x * VPN_y - VPN_x * VUP_y) / u_value;

        // v = VPN X u / |VPN X u|
        v_value =
          Math.pow((VPN_y * u_z - u_y * VPN_z), 2) +
          Math.pow((u_x * VPN_z - VPN_x * u_z), 2) +
          Math.pow((VPN_x * u_y - u_x * VPN_y), 2);
        v_value = Math.sqrt(v_value);

        v_x = (VPN_y * u_z - u_y * VPN_z) / v_value;
        v_y = (u_x * VPN_z - VPN_x * u_z) / v_value;
        v_z = (VPN_x * u_y - u_x * VPN_y) / v_value;

        R[0][0] = u_x;
        R[1][0] = u_y;
        R[2][0] = u_z;
        R[3][0] = 0.0;

        R[0][1] = v_x;
        R[1][1] = v_y;
        R[2][1] = v_z;
        R[3][1] = 0.0;

        R[0][2] = n_x;
        R[1][2] = n_y;
        R[2][2] = n_z;
        R[3][2] = 0.0;

        R[0][3] = 0.0;
        R[1][3] = 0.0;
        R[2][3] = 0.0;
        R[3][3] = 1.0;

        // Because R is an orthogonal matrix,invertible matrix of R is equal to R's
        // transposed matrix

        // Matrix of T
        T[0][0] = 1.0;
        T[1][0] = 0.0;
        T[2][0] = 0.0;
        T[3][0] = -VRP_x;

        T[0][1] = 0.0;
        T[1][1] = 1.0;
        T[2][1] = 0.0;
        T[3][1] = -VRP_y;

        T[0][2] = 0.0;
        T[1][2] = 0.0;
        T[2][2] = 1.0;
        T[3][2] = -VRP_z;

        T[0][3] = 0.0;
        T[1][3] = 0.0;
        T[2][3] = 0.0;
        T[3][3] = 1.0;

        if (mark == "mcw") {
            // get the invertible matrix of T
            T[3][0] *= (-1);
            T[3][1] *= (-1);
            T[3][2] *= (-1);

            // get the invertible matrix of R
            R[1][0] = v_x;
            R[2][0] = n_x;

            R[0][1] = u_y;
            R[1][1] = v_y;
            R[2][1] = n_y;

            R[0][2] = u_z;
            R[1][2] = v_z;

            // calculation of R(inverse)*T(inverse)
            M = calc.MatrixTimes(T, R);
        }
        // calculation of R*T
        if (mark == "mwc")
            M = calc.MatrixTimes(R, T);
        return M;
    }

    public static double[][] Mwc(
      double VRP_x, double VRP_y, double VRP_z,
      double VPN_x, double VPN_y, double VPN_z,
      double VUP_x, double VUP_y, double VUP_z) {
        double M[][] = new double[4][4];
        M = get_matrix(VRP_x, VRP_y, VRP_z,
          VPN_x, VPN_y, VPN_z,
          VUP_x, VUP_y, VUP_z, "mwc");
        return M;
    }

    public static double[][] Mcw(
      double VRP_x, double VRP_y,
      double VRP_z, double VPN_x,
      double VPN_y, double VPN_z,
        double VUP_x, double VUP_y, double VUP_z) {
        double M[][] = new double[4][4];
        M = get_matrix(VRP_x, VRP_y, VRP_z,
          VPN_x, VPN_y, VPN_z,
          VUP_x, VUP_y, VUP_z, "mcw");
        return M;
    }

}
```

### MyModel

```java
// Initialize the global data structures;
public class MyModel {
    /* definition of the image buffer */
    public static final int ROWS = 512;
    public static final int COLS = 512;
    public static int image[][] = new int[ROWS][COLS];
    /* definition of window on the image plane in the camera coordinates */
    /* They are used in mapping (j, i) in the screen coordinates into */
    /* (x, y) on the image plane in the camera coordinates */
    /* The window size used here simulates the 35 mm film. */
    public static double xmin = 0.0175;
    public static double ymin = -0.0175;
    public static double xmax = -0.0175;
    public static double ymax = 0.0175;
    /* definition of the camera parameters */
    public static double VRP[] = new double[] {
        1.0,
        2.0,
        3.5
    };
    public static double VPN[] = new double[] {
        0.0,
        -1.0,
        -2.5
    };
    public static double VUP[] = new double[] {
        0.0,
        1.0,
        0.0
    };
    /* focal length simulating 50 mm lens */
    public static double focal = 0.05;
    /* definition of light source */
    public static double LPR[] = new double[] {
        -10.0, 10.0, 2.0
    };
    /* intensity of the point light source */
    public static double IP = 200.0;
    public static double Mwc[][] = new double[4][4];
    public static double Mcw[][] = new double[4][4];
    public static double[] P0 = new double[4];
    public static double[] V0 = new double[4];
    /* create a spherical object */
    public static SPHERE obj1 = new SPHERE(1.0, 1.0, 1.0, 1.0, 0.75);
    /* create a polygon object */
    public static POLY4 obj2 = new POLY4();
}
```

### Poly4

```java
/* Definition of Polygon with 4 edges */
public class POLY4 {
    double[][] v = new double[4][3];
    double[] N = new double[3];
    double kd;
    public POLY4() {
        /* v0 */
        v[0][0] = 0.0;
        v[0][1] = 0.0;
        v[0][2] = 0.0;

        /* v1 */
        v[1][0] = 0.0;
        v[1][1] = 0.0;
        v[1][2] = 2.0;

        /* v2 */
        v[2][0] = 2.0;
        v[2][1] = 0.0;
        v[2][2] = 2.0;

        /* v3 */
        v[3][0] = 2.0;
        v[3][1] = 0.0;
        v[3][2] = 0.0;

        /* normal of the polygon */
        N[0] = 0.0;
        N[1] = 1.0;
        N[2] = 0.0;

        /* diffuse reflection coefficient */
        kd = 0.8;
    }
}
```

### Sphere

```java
/* Definition of the structure for Sphere */
public class SPHERE {
    // define
    /* x y z as center of the circle */
    /* radius as radius of the circle */
    /* kd as diffuse reflection coefficient */
    double x, y, z;
    double radius;
    double kd;
    public SPHERE(double x, double y, double z, double radius, double kd) {
        super();
        this.x = x;
        this.y = y;
        this.z = z;
        this.radius = radius;
        this.kd = kd;
    }
}
```

### Ray Construction

```java
public class ray_construction {

    static int i;
    static int j;
    static double[] P0 = new double[4];
    static double[] V0 = new double[4];

    // Construction of ray
    // Input: pixel index (i, j) in the screen coordinates
    // Output: P0, V0 (for parametric ray equation P = P0 + V0*t)
    // in the world coordinates.
    public static void generate(int j, int i) {

        // Step 1:
        // Map (j, i) in the screen coordinates to (xc, yc) in the
        // camera coordinates.

        double xc, yc;

        xc = j * (MyModel.xmax - MyModel.xmin) / (MyModel.COLS - 1) + MyModel.xmin;
        yc = i * (MyModel.ymax - MyModel.ymin) / (MyModel.ROWS - 1) + MyModel.ymin;

        // Step 2:
        // Transform the origin (0.0, 0.0, 0.0) of the camera
        // coordinates to P0 in the world coordinates using the
        // transformation matrix Mcw. Note that the transformed result
        // should be VRP.

        double[] origin_camera = {
            0.0,
            0.0,
            0.0,
            1.0
        };
        P0 = calc.transform(MyModel.Mcw, origin_camera);

        // Step 3:
        // Transform the point (xc, yc, f) on the image plane in
        // the camera coordinates to P1 in the world coordinates using
        // the transformation matrix Mcw.

        double[] data = new double[4];
        double[] P1 = new double[4];

        data[0] = xc;
        data[1] = yc;
        data[2] = MyModel.focal;
        data[3] = 1;
        P1 = calc.transform(MyModel.Mcw, data);

        // V0 = P1 – P0;
        V0[0] = P1[0] - P0[0];
        V0[1] = P1[1] - P0[1];
        V0[2] = P1[2] - P0[2];

        // normalize V0 into unit length;
        V0 = calc.normalize(V0);
        MyModel.P0 = P0;
        MyModel.V0 = V0;
    }
}
```

### Ray Object Intersection

```java
public class ray_object_intersection {

    static double C;
    static double[] P0 = new double[4];
    static double[] V0 = new double[4];
    static double[] P = new double[4];

    static double[] N = new double[3];
    static double[] N1 = new double[3];
    static double[] N2 = new double[3];

    static double kd;
    static double kd1;
    static double kd2;

    public static boolean main(double[] P0, double[] V0) {

        double t1 = 0;
        double t2 = 0;

        kd1 = MyModel.obj1.kd;
        kd2 = MyModel.obj2.kd;

        t1 = ray_sphere_intersection(P0, V0);
        t2 = ray_polygon_intersection(P0, V0);

        if (t1 == 0 && t2 == 0)
            return false;
        else if (t2 == 0) {
            shading.P = calc.vector_add(P0, calc.vector_times(V0, t1));
            shading.N = N1;
            shading.kd = kd1;
            return true;
        } else if (t1 == 0) {
            shading.P = calc.vector_add(P0, calc.vector_times(V0, t2));
            shading.N = N2;
            shading.kd = kd2;
            return true;
        } else if (t1 < t2) {
            shading.P = calc.vector_add(P0, calc.vector_times(V0, t1));
            shading.N = N1;
            shading.kd = kd1;
            return true;
        } else {
            shading.P = calc.vector_add(P0, calc.vector_times(V0, t2));
            shading.N = N2;
            shading.kd = kd2;
            return true;
        }
    }

    public static double ray_sphere_intersection(double[] P0, double[] V0) {

        double t, A, B, C, R, delta;
        double t1, t2;

        // using method 2 from notes
        // solving A^2+B^2+C^2=R^2 by PO+t*V0

        R = MyModel.obj1.radius;
        A = V0[0] * V0[0] + V0[1] * V0[1] + V0[2] * V0[2];
        B = (P0[0] * V0[0] + P0[1] * V0[1] + P0[2] * V0[2]) * 2;
        C = P0[0] * P0[0] + P0[1] * P0[1] + P0[2] * P0[2] - R * R;

        delta = B * B - 4 * A * C;

        // determine if the ray has a intersection with the sphere
        if (delta < 0)
            return 0;
        else {
            t1 = (Math.abs(Math.sqrt(delta)) - B) / (2 * A);
            t2 = (-Math.abs(Math.sqrt(delta)) - B) / (2 * A);
            if (t1 < t2)
                t = t1;
            else
                t = t2;
            N1[0] = (P0[0] + V0[0] * t - MyModel.obj1.x);
            N1[1] = (P0[1] + V0[1] * t - MyModel.obj1.y);
            N1[2] = (P0[2] + V0[2] * t - MyModel.obj1.z);
            return t;
        }
    }

    public static double ray_polygon_intersection(double[] P0, double[] V0) {

        double N_value;
        double D;
        double t;

        double[] V1 = new double[3];
        double[] V2 = new double[3];
        double[] P = new double[3];

        V1[0] = MyModel.obj2.v[1][0] - MyModel.obj2.v[0][0];
        V1[1] = MyModel.obj2.v[1][1] - MyModel.obj2.v[0][1];
        V1[2] = MyModel.obj2.v[1][2] - MyModel.obj2.v[0][2];

        V2[0] = MyModel.obj2.v[2][0] - MyModel.obj2.v[0][0];
        V2[1] = MyModel.obj2.v[2][1] - MyModel.obj2.v[0][1];
        V2[2] = MyModel.obj2.v[2][2] - MyModel.obj2.v[0][2];

        N_value = Math.sqrt(Math.pow((V1[1] * V2[2] - V2[1] * V1[2]), 2) +
                  Math.pow((V2[0] * V1[2] - V1[0] * V2[2]), 2) +
                  Math.pow((V1[0] * V2[1] - V2[0] * V1[1]), 2));
        N[0] = (V1[1] * V2[2] - V2[1] * V1[2]) / N_value;
        N[1] = (V2[0] * V1[2] - V1[0] * V2[2]) / N_value;
        N[2] = (V1[0] * V2[1] - V2[0] * V1[1]) / N_value;


        // compute D

        D = (N[0] * MyModel.obj2.v[0][0] +
              N[1] * MyModel.obj2.v[0][1] +
              N[2] * MyModel.obj2.v[0][2]) * (-1);

        t = ((N[0] * P0[0] + N[1] * P0[1] + N[2] * P0[2]) + D) /
        (N[0] * V0[0] + N[1] * V0[1] + N[2] * V0[2]) * (-1);

        P[0] = P0[0] + V0[0] * t;
        P[1] = P0[1] + V0[1] * t;
        P[2] = P0[2] + V0[2] * t;

        if (is_inside(P, N)) {
            N2 = N;
            return t;
        } else
            return 0;
    }

    public static boolean is_inside(double[] P, double[] N) {

        int number = 0;
        int i;
        int x, y;
        double k, b;
        double[][] V = new double[5][2];
        int[] remain = new int[2];
        remain = drop(N);
        x = remain[0];
        y = remain[1];

        // drop one dimension to make the scene into 2D
        V[0][0] = MyModel.obj2.v[0][x];
        V[0][1] = MyModel.obj2.v[0][y];
        V[1][0] = MyModel.obj2.v[1][x];
        V[1][1] = MyModel.obj2.v[1][y];
        V[2][0] = MyModel.obj2.v[2][x];
        V[2][1] = MyModel.obj2.v[2][y];
        V[3][0] = MyModel.obj2.v[3][x];
        V[3][1] = MyModel.obj2.v[3][y];

        V[0][0] = V[0][0] - P[x];
        V[0][1] = V[0][1] - P[y];
        V[1][0] = V[1][0] - P[x];
        V[1][1] = V[1][1] - P[y];
        V[2][0] = V[2][0] - P[x];
        V[2][1] = V[2][1] - P[y];
        V[3][0] = V[3][0] - P[x];
        V[3][1] = V[3][1] - P[y];
        V[4][0] = V[0][0];
        V[4][1] = V[0][1];

        //calculate the y value of the point intersected with x axis
        for (i = 0; i <= 3; i++) {
            k = (V[i][1] - V[i + 1][1]) / (V[i][0] - V[i + 1][0]);
            b = V[i][1] - V[i][0] * k;
            // y = kx +b
            if ((-b / k) >= 0)
                number++;
        }
        if (number % 2 == 1)
            return true;
        else
            return false;
    }

    public static int[] drop(double[] N) {
        int[] r = new int[2];
        double max;
        int k = 0;
        max = calc.max(N[0], N[1], N[2]);
        for (int i = 0; i < 3; i++) {
            if (N[i] != max) {
                r[k] = i;
                k++;
            }
        }
        return r;
    }
}
```

### Handle Ray Tracing

```java
public class ray_tracing {
    static double C;
    static double[] P0 = new double[4];
    static double[] V0 = new double[4];
    static double kd;
    public static double main(double[] P0, double[] V0) {
        boolean found;
        found = ray_object_intersection.main(P0, V0);
        if (found) {
            C = shading.get();
            return C;
        } else {
          return 0;
        }
    }
}
```

### Shading

```java
// Shading:
// Input: P[3] – point position
// N[3] – surface normal at that point
// kd – diffuse reflection coefficient of the surface
// Output: C – shading value
public class shading {
    public static double[] P = new double[3];
    public static double[] N = new double[3];
    public static double kd;
    public static double C;
    public static double[] L = new double[4]; // LPR is the light position
    public static double get() {
        // L = LPR - P;
        L[0] = MyModel.LPR[0] - P[0];
        L[1] = MyModel.LPR[1] - P[1];
        L[2] = MyModel.LPR[2] - P[2];
        // normalize L to unit length;
        L = calc.normalize(L);
        C = MyModel.IP * kd * (N[0] * L[0] + N[1] * L[1] + N[2] * L[2]);
        return C;
    }
}
```

### Utils

```java
public class calc {
    public static double[][] MatrixTimes(double[][] a, double[][] b) {
        double[][] c = new double[4][4];
        int i, j, k;
        double s;
        for (j = 0; j <= 3; j++) {
            for (i = 0; i <= 3; i++) {
                s = 0;
                for (k = 0; k <= 3; k++) {
                    s += a[k][j] * b[i][k];
                }
                c[i][j] = s;
            }
        }
        return c;
    }

    public static double[] transform(double[][] M, double[] data) {
        int i, j;
        double s;
        double[] newdata = new double[4];
        for (j = 0; j <= 3; j++) {
            s = 0;
            for (i = 0; i <= 3; i++) {
                s += M[i][j] * data[i];
            }
            newdata[j] = s;
        }
        return newdata;
    }

    public static double[] vector_minus(double[] a, double[] b) {
        double[] c = new double[4];
        c[0] = a[0] - b[0];
        c[1] = a[1] - b[1];
        c[2] = a[2] - b[2];
        c[3] = a[3] - b[3];
        return c;
    }

    public static double[] vector_add(double[] a, double[] b) {
        double[] c = new double[4];
        c[0] = a[0] + b[0];
        c[1] = a[1] + b[1];
        c[2] = a[2] + b[2];
        c[3] = a[3] + b[3];
        return c;
    }

    public static double[] normalize(double[] c) {
        double d;
        d = Math.sqrt(c[0] * c[0] + c[1] * c[1] + c[2] * c[2]);
        c[0] = c[0] / d;
        c[1] = c[1] / d;
        c[2] = c[2] / d;
        return c;
    }

    public static double[] vector_times(double[] d, double t) {
        d[0] *= t;
        d[1] *= t;
        d[2] *= t;
        return d;
    }

    public static double dot_product(double[] a, double[] b) {
        double c;
        c = a[0] * b[0] + a[1] * b[1] + a[2] * b[2];
        return c;
    }

    public static double max(double a, double b, double c) {
        double d = 0;
        if (a > d)
            d = a;
        if (b > d)
            d = b;
        if (c > d)
            d = c;
        return d;
    }
}
```

## Volume Rendering

Additional Files:

### Ray Box Intersection

```java
//Ray-Box Intersection
//Input: ray – P0, V0
//Output: n – the number of intersections found
// n = 0, no intersection
// n = 1, one intersection
// n = 2, two intersections
// It will be very rare to find more two intersections.
// In this case, you can set it to 0.
// ts[2] stores the found intersections.
// if n = 2, you should have ts[0] < ts[1].
// kd, the diffuse reflection coefficient of the surface.

public class ray_box_intersection {
    public static int main() {
        int n = 0;
        double t;
        double t0 = 0, t1 = 0;
        double x, y, z;
        double[] intersection = new double[4];

        // for side x = 0
        t = (0 - global.P0[0]) / global.V0[0];
        y = global.P0[1] + global.V0[1] * t;
        z = global.P0[2] + global.V0[2] * t;
        if (y >= 0 && y <= global.ROWS && z >= 0 && z <= global.LAYS) {
            intersection[n] = t;
            n++;
        }

        // for side x = 127
        t = (127 - global.P0[0]) / global.V0[0];
        y = global.P0[1] + global.V0[1] * t;
        z = global.P0[2] + global.V0[2] * t;
        if (y >= 0 && y <= global.ROWS && z >= 0 && z <= global.LAYS) {
            intersection[n] = t;
            n++;
        }

        // for side y = 0
        t = (0 - global.P0[1]) / global.V0[1];
        x = global.P0[0] + global.V0[0] * t;
        z = global.P0[2] + global.V0[2] * t;
        if (x >= 0 && x <= global.COLS && z >= 0 && z <= global.LAYS) {
            intersection[n] = t;
            n++;
        }

        // for side y = 127
        t = (127 - global.P0[1]) / global.V0[1];
        x = global.P0[0] + global.V0[0] * t;
        z = global.P0[2] + global.V0[2] * t;
        if (x >= 0 && x <= global.COLS && z >= 0 && z <= global.LAYS) {
            intersection[n] = t;
            n++;
        }

        // for side z = 0
        t = (0 - global.P0[2]) / global.V0[2];
        x = global.P0[0] + global.V0[0] * t;
        y = global.P0[1] + global.V0[1] * t;
        if (x >= 0 && x <= global.COLS && y >= 0 && y <= global.ROWS) {
            intersection[n] = t;
            n++;
        }

        // for side z = 127
        t = (127 - global.P0[2]) / global.V0[2];
        x = global.P0[0] + global.V0[0] * t;
        y = global.P0[1] + global.V0[1] * t;
        if (x >= 0 && x <= global.COLS && y >= 0 && y <= global.ROWS) {
            intersection[n] = t;
            n++;
        }

        // find more than two intersections
        if (n > 2) {
            n = 0;
        }

        // intersections found
        if (n != 0) {
            if (intersection[0] < intersection[1]) {
                t0 = intersection[0];
                t1 = intersection[1];
            } else {
                t1 = intersection[0];
                t0 = intersection[1];
            }
            global.ts[0] = t0;
            global.ts[1] = t1;
        }
        return n;
    }
}
```

### Trilinear Interpolation

```java
public class trilinear_interpolation {
    public static int x0, x1, y0, y1, z0, z1;
    public static double xd, yd, zd;
    public static double c000, c100, c001, c101, c010, c110, c011, c111;
    public static double c00, c01, c10, c11;
    public static double c0, c1;
    public static double c;
    public static double s, t;
    public static double main(int[][][] data, double x, double y, double z, String mark) {

        x0 = (int) x;
        x1 = x0 + 1;
        y0 = (int) y;
        y1 = y0 + 1;
        z0 = (int) z;
        z1 = z0 + 1;

        // return 0 if the point is outside the boundary
        if (x1 > 127 || x0 < 0 || y1 > 127 || y0 < 0 || z1 > 127 || z0 < 0)
            return 0;

        c000 = data[z1][y0][x0];
        c100 = data[z1][y0][x1];

        c001 = data[z1][y1][x0];
        c101 = data[z1][y1][x1];

        c010 = data[z0][y0][x0];
        c110 = data[z0][y0][x1];

        c011 = data[z0][y1][x0];
        c111 = data[z0][y1][x1];

        s = x - x0;
        t = 1.0 - s;

        c00 = c000 * t + c100 * s;
        c01 = c001 * t + c101 * s;
        c10 = c010 * t + c110 * s;
        c11 = c011 * t + c111 * s;

        s = z - z0;
        t = 1 - s;

        c0 = c10 * t + c00 * s;
        c1 = c11 * t + c01 * s;

        s = y - y0;
        t = 1 - s;
        c = c0 * t + c1 * s;

        if (mark == "opacity")
            return c / 255; // opacity being normalized in the range [0.0, 1.0]
        else
            return c;
    }
}
```

### Volumn Ray Tracing

```java
//Main Volume Ray-Tracing Function:
//Input:
// O – starting point of the ray
// V – unit-length vector pointing in the direction of the ray
// ts[0] and ts[1] are two points on the ray. Assuming
// ts[0] < ts[1]
//Output: the integrated shading value along the ray between
// ts[0] and ts[1]

public class volume_ray_tracing {
    public static int main() {

        double t, t0, t1;
        double x, y, z;
        double a, c;
        double dt = 1.0; // the interval for sampling along the ray
        double C = 0.0; // for accumulating the shading value
        double T = 1.0; // for accumulating the transparency

        t0 = global.ts[0];
        t1 = global.ts[1];

        /* Marching through the CT volume from t0 to t1 by step size dt. */
        /* Compute the 3D coordinates of the current sample position in the volume */

        for (t = t0; t <= t1; t += dt) {
            x = global.P0[0] + t * global.V0[0];
            y = global.P0[1] + t * global.V0[1];
            z = global.P0[2] + t * global.V0[2];
            /*
             * Obtain the shading value C and opacity value A from the shading volume and CT
             * volume, respectively, by using tri-linear interpolation.
             */
            // obtain opacity(density) value
            a = trilinear_interpolation.main(global.CT, x, y, z, "opacity");
            // obtain shading value c
            c = trilinear_interpolation.main(global.SHADING, x, y, z, "shading");

            /* Accumulate the shading values in the front-to-back order. */
            C += T * a * c;
            T *= (1.0 - a);
        }
        return (int) C;
    }
}
```
