# GCP Load Balancer Automation

This workflow automates the creation and destruction of Google Cloud Load Balancers using Terraform.

## 📋 Workflow Inputs

When triggering the workflow, you will be prompted to provide the following inputs:

| Field | Required | Description | Example |
| :--- | :--- | :--- | :--- |
| **`region`** | Yes | GCP region used by Terraform resources | `us-central1` |
| **`lb_name`** | Yes | Unique LB name. Lowercase, numbers, hyphens only | `my-app-lb` |
| **`default_backend_service_name`** | Yes | Backend service used for unmatched traffic | `frontend-backend-svc` |
| **`backend_a_service_name`** | No | Optional backend service A | `api-backend-svc` |
| **`backend_a_path_prefix`** | No | Path routed to Backend A | `/api` |
| **`backend_b_service_name`** | No | Optional backend service B | `admin-backend-svc` |
| **`backend_b_path_prefix`** | No | Path routed to Backend B | `/admin` |
| **`tf_action`** | Yes | Terraform action | `plan`, `apply`, `destroy` |

---

## 🚀 How to Create a Load Balancer

1. Go to the **Actions** tab in your GitHub repository.
2. On the left sidebar, click on the **GCP LB automation** workflow.
3. On the right side, click the **Run workflow** dropdown button.
4. Fill out the required fields from the table above.
5. Set the **`tf_action`** to **`apply`**.
6. Click the green **Run workflow** button to start the deployment.

---

## 🗑️ How to Destroy a Load Balancer

1. Go to the **Actions** tab and select the **GCP LB automation** workflow.
2. Click the **Run workflow** dropdown button.
3. Enter the **exact same `lb_name`** and **`region`** that you used when creating the Load Balancer.
4. Set the **`tf_action`** to **`destroy`**.
5. Click the green **Run workflow** button to tear down the infrastructure.
