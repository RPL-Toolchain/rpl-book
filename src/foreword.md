# Foreword

Rust has gained significant popularity as a system programming language, valued for its capacity to guarantee memory safety through compile-time verification without sacrificing runtime performance. Since system programming must sometimes perform low-level tasks, Rust also provides unsafe code to bypass the compiler's safety checks, shifting the responsibility for ensuring memory safety to the developer.Therefore, linters are developed to automatically detect common errors and potential misuses of unsafe code. The common approach for existing Rust linters is to embed the checking logic directly into their implementation, which limits their customizability and extensibility. To address this limitation, we introduce RPL, an extensible and customizable Rust linter. The key feature of RPL is the decoupling of checking rules' definition from their detection logic. In particular, RPL consists of two primary components: a Domain-Specific Language (DSL) that allows developers to model/define code patterns, and a detection engine to detect instances of these patterns.

While RPL is still under active development, we intend for this document to serve as a comprehensive guide for future contributors and anyone interested in this work.

_Carefully we wipe them hour by hour, And let no dust alight._

— stuuupidcat, 2025-04-24, Beijing
