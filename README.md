import subprocess
import json
import sys
from datetime import datetime

REPORT_FILE = "scan_report.txt"

def run_command(command, tool_name):
    print(f"\n[+] Running {tool_name}...")
    
    try:
        result = subprocess.run(
            command,
            shell=True,
            capture_output=True,
            text=True
        )

        output = result.stdout + result.stderr

        with open(REPORT_FILE, "a") as report:
            report.write(f"\n\n===== {tool_name} =====\n")
            report.write(output)

        print(output)

        return result.returncode

    except Exception as e:
        print(f"[-] Error running {tool_name}: {e}")
        return 1


def check_failure(return_code, tool_name):
    if return_code != 0:
        print(f"\n[-] {tool_name} found issues!")
        return True
    return False


def main():

    print("=" * 60)
    print(" CI/CD Automated Security Scanning ")
    print("=" * 60)

    with open(REPORT_FILE, "w") as report:
        report.write(f"Security Scan Report\n")
        report.write(f"Generated: {datetime.now()}\n")
        report.write("=" * 60 + "\n")

    failed = False

    # -----------------------------------------
    # 1. SAST Scan - Semgrep
    # -----------------------------------------
    sast_cmd = "semgrep --config=auto ."
    rc = run_command(sast_cmd, "SAST Scan - Semgrep")

    if check_failure(rc, "Semgrep"):
        failed = True

    # -----------------------------------------
    # 2. Dependency Scan - Trivy Filesystem
    # -----------------------------------------
    dep_cmd = "trivy fs --severity HIGH,CRITICAL ."
    rc = run_command(dep_cmd, "Dependency Scan - Trivy")

    if check_failure(rc, "Trivy Dependency Scan"):
        failed = True

    # -----------------------------------------
    # 3. Secret Scan - Gitleaks
    # -----------------------------------------
    secret_cmd = "gitleaks detect --source ."
    rc = run_command(secret_cmd, "Secret Scan - Gitleaks")

    if check_failure(rc, "Gitleaks"):
        failed = True

    # -----------------------------------------
    # 4. Docker Image Scan
    # -----------------------------------------
    image_name = "myapp:latest"

    container_cmd = f"trivy image --severity HIGH,CRITICAL {image_name}"
    rc = run_command(container_cmd, "Container Scan - Trivy")

    if check_failure(rc, "Container Scan"):
        failed = True

    # -----------------------------------------
    # Final Status
    # -----------------------------------------
    print("\n" + "=" * 60)

    if failed:
        print("[-] Security scan failed!")
        print("[-] Blocking deployment pipeline.")
        sys.exit(1)

    else:
        print("[+] All security scans passed!")
        print("[+] Deployment can continue.")
        sys.exit(0)


if __name__ == "__main__":
    main()
