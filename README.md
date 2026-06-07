import os
import hashlib
from pathlib import Path


class DuplicateFinder:

    def __init__(self, root_path):
        self.root_path = Path(root_path)
        self.hashes = {}
        self.duplicates = []

    def calculate_hash(self, file_path):
        sha256 = hashlib.sha256()

        try:
            with open(file_path, "rb") as f:
                for chunk in iter(lambda: f.read(8192), b""):
                    sha256.update(chunk)

            return sha256.hexdigest()

        except Exception:
            return None

    def scan(self):
        print("Scanning files...\n")

        for root, _, files in os.walk(self.root_path):

            for file in files:

                path = Path(root) / file

                file_hash = self.calculate_hash(path)

                if not file_hash:
                    continue

                if file_hash in self.hashes:

                    self.duplicates.append(
                        (
                            self.hashes[file_hash],
                            path
                        )
                    )

                else:
                    self.hashes[file_hash] = path

    def report(self):

        if not self.duplicates:
            print("No duplicate files found.")
            return

        print(f"\nFound {len(self.duplicates)} duplicate files:\n")

        for original, duplicate in self.duplicates:

            print("=" * 70)
            print(f"Original : {original}")
            print(f"Duplicate: {duplicate}")

    def duplicate_size(self):

        total = 0

        for _, duplicate in self.duplicates:

            try:
                total += duplicate.stat().st_size
            except:
                pass

        return total


def human_size(size):

    for unit in ["B", "KB", "MB", "GB", "TB"]:

        if size < 1024:
            return f"{size:.2f} {unit}"

        size /= 1024

    return f"{size:.2f} PB"


def main():

    print("=" * 50)
    print("Duplicate File Finder")
    print("=" * 50)

    directory = input(
        "\nEnter directory path: "
    ).strip()

    finder = DuplicateFinder(directory)

    finder.scan()

    finder.report()

    print(
        "\nPotential space savings:",
        human_size(finder.duplicate_size())
    )


if __name__ == "__main__":
    main()
