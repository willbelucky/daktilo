---
layout: post
title:  "PCA: Relationship of matrices"
subtitle: "Loading matrix & Eigen value & Eigen vector"
date:   2018-05-10 23:00:00
categories: [statistics]
---

# Loading Matrix
$$\xi_i$$ is i-th principal component. \\
$$
    \begin{matrix}
            & \xi_1            & \xi_2            & \xi_3            \\
        X_1 & corr(\xi_1, X_1) & corr(\xi_2, X_1) & corr(\xi_3, X_1) \\
        X_2 & corr(\xi_1, X_2) & corr(\xi_2, X_2) & corr(\xi_3, X_2) \\
        X_3 & corr(\xi_1, X_3) & corr(\xi_2, X_3) & corr(\xi_3, X_3) \\
    \end{matrix}
$$

## Loading Matrix is orthogonal.
$$
    corr(\xi_1, X_1)corr(\xi_2, X_1) + corr(\xi_1, X_2)corr(\xi_2, X_2) + corr(\xi_1, X_3)corr(\xi_2, X_3) = 0 \\
    corr(\xi_2, X_1)corr(\xi_3, X_1) + corr(\xi_2, X_2)corr(\xi_3, X_2) + corr(\xi_2, X_3)corr(\xi_3, X_3) = 0 \\
    corr(\xi_3, X_1)corr(\xi_1, X_1) + corr(\xi_3, X_2)corr(\xi_1, X_2) + corr(\xi_3, X_3)corr(\xi_1, X_3) = 0
$$

# Eigen value
$$\lambda_i$$ is an eigen value of $$\xi_i$$. \\
$$\lambda_i = Var(\xi_i) = corr(\xi_i, X_1)^2 + corr(\xi_i, X_2)^2 + ... + corr(\xi_i, X_p)^2$$ \\
$$
    \begin{matrix}
                & \xi_1     & \xi_2     & \xi_3     \\
        \lambda & \lambda_1 & \lambda_2 & \lambda_3 \\
    \end{matrix}
$$

# Eigen vector
$$\beta_{ij}$$ is an eigen vector of $$\xi_i$$ and $$std(X_j)$$. \\
$$
    \begin{matrix}
                 & \xi_1                                     & \xi_2                                     & \xi_3                                     \\
        std(X_1) & \frac{corr(\xi_1, X_1)}{\sqrt{\lambda_1}} & \frac{corr(\xi_2, X_1)}{\sqrt{\lambda_2}} & \frac{corr(\xi_3, X_1)}{\sqrt{\lambda_3}} \\
        std(X_2) & \frac{corr(\xi_1, X_2)}{\sqrt{\lambda_1}} & \frac{corr(\xi_2, X_2)}{\sqrt{\lambda_2}} & \frac{corr(\xi_3, X_2)}{\sqrt{\lambda_3}} \\
        std(X_3) & \frac{corr(\xi_1, X_3)}{\sqrt{\lambda_1}} & \frac{corr(\xi_2, X_3)}{\sqrt{\lambda_2}} & \frac{corr(\xi_3, X_3)}{\sqrt{\lambda_3}} \\
    \end{matrix} =
    \begin{matrix}
                 & \xi_1      & \xi_2      & \xi_3      \\
        std(X_1) & \beta_{11} & \beta_{21} & \beta_{31} \\
        std(X_2) & \beta_{12} & \beta_{22} & \beta_{32} \\
        std(X_3) & \beta_{13} & \beta_{23} & \beta_{33} \\
    \end{matrix}
$$

## Eigen vector is orthonormal.

### Eigen vector is orthogonal.
$$
    \beta_{11}\beta_{21} + \beta_{12}\beta_{22} + \beta_{13}\beta_{23} = 0 \\
    \beta_{21}\beta_{31} + \beta_{22}\beta_{32} + \beta_{23}\beta_{33} = 0 \\
    \beta_{31}\beta_{11} + \beta_{32}\beta_{12} + \beta_{33}\beta_{13} = 0
$$

### Eigen vector is a unit vector.
$$
    \beta_{11}^2 + \beta_{12}^2 + \beta_{13}^2 = 1 \\
    \beta_{21}^2 + \beta_{22}^2 + \beta_{23}^2 = 1 \\
    \beta_{31}^2 + \beta_{32}^2 + \beta_{33}^2 = 1
$$

## Simple linear regression $$X_1$$ on $$\xi_1, \xi_2, \xi_3$$
Because no information is lost by using PCA,
$$
    X_1 = \beta_{11}\xi_1 + \beta_{21}\xi_2 + \beta_{31}\xi_3
$$
So, R-square of the regression is 1 and RMSE is 0.