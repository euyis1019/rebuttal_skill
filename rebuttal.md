# Rebuttal to Reviewer JwJA

我们感谢审稿人对本文的认真阅读和宝贵建议。以下针对审稿人提出的各项**缺点（Weaknesses）/问题**逐一回应。

## **缺点 1 ：评估有限，缺乏多随机种子实验**

感谢审稿人提出这一重要问题。实际上，我们在补充材料C.1节中已报告了基于4个随机种子的实验，并在Figure 1中展示了成功率及其误差条。结果表明，无论是原始 WebGRPO 还是增强版本 WebGRPO（+Skills），在两个模型系列中均显著优于所有基线方法。此外，我们进一步补充了使用5个随机种子的实验以进一步验证方法的稳定性。以下是各方法在 WebArena-Lite 上的平均成功率：

- **GLM-4：**SFT 22.2%，Filtered BC 24.5%，AWR 27.6%，DigiRL 31.4%，WEBRL 43.4%，WebGRPO 50.6%，WebGRPO (+Skills) 58.2%。  
- **LLaMA3.1：**SFT 20.3%，Filtered BC 23.1%，AWR 28.4%，DigiRL 30.0%，WEBRL 41.9%，WebGRPO 51.6%，WebGRPO (+Skills) 59.7%。  

结果进一步验证了我们的方法在多个随机初始化下均表现稳定，并取得显著性能提升。

## **缺点2 ：缺乏基础模型规模分析**

感谢审稿人指出这一问题。为进一步分析基础模型规模对性能的影响，我们补充了在 Qwen2.5-3B 模型上的实验，并使用 5 个随机种子进行评估。平均成功率为：Reddit 36.3%，GitLab 51.2%，CMS 47.3%，Map 32.3%，OSS 28.1%，整体平均为 39.0%，表明我们的方法在小模型上依然有效。后续我们计划在  Llama3.1-70B 规模的模型上开展进一步探索，以系统评估规模扩展对表现的影响。

## **缺点3 ：奖励/进化模型规模分析同样有限**

感谢审稿人对 RXERM 奖励机制评估的关注。我们澄清，RXERM 包含两个部分：执行奖励（判断任务是否完成）和推理奖励（评估中间推理步骤的质量）。我们在早期已对 GPT‑4o 作为评估器进行了初步验证。具体而言，我们在多个任务上随机抽取了 100 条轨迹，将 GPT‑4o 的判断与人工标注结果以及其他评估模型（Claude、DeepSeek‑R1 等）进行了对比。结果显示，GPT‑4o 与人工标注的一致率较高（执行奖励 >90%，推理奖励 >85%），且与其他强评估器之间差异很小（<3%）。基于这一稳定性结果，并考虑到我们拥有 GPT‑4o 的公司内部 API 可免费使用，从成本和资源角度出发，最终选择 GPT‑4o 作为 RXERM 的评估器。此外，关于 SkillGenesis 在不同 LLM 下的性能影响，我们已在补充材料 C.4 Performance Comparison of SkillEvo with Different LLMs 中给出了系统分析。为回应审稿人的关切，我们将在修订版中进一步补充实验：使用多个开源评估模型替换 GPT‑4o，对 RXERM 的执行奖励准确性和推理奖励可靠性进行对比分析，从而验证方法的鲁棒性。

## **缺点4 ：缺乏数据集和最终模型训练细节**。

感谢审稿人提出该问题。**关于数据集：**SFT（监督微调，Supervised Fine-Tuning）训练所使用的数据集是 WebArena-Lite 提供的一个人类标注演示数据集。该数据集由 5 位作者基于约 40 个任务原型精心设计并生成，最终构建出 1,186 条高质量的专家轨迹（expert demonstrations）。这些演示覆盖了WebArena-Lite的5个真实网站，任务类型多样，操作步骤清晰。对于强化学习阶段的训练数据，正文第 3.1 节的 Baselines 部分已有明确描述。我们采用了自动任务生成方式，具体来说，指令来自于一个技能提议器（skill proposer）。该模块在训练过程中会基于初始网页状态（包含5个网站）自动生成大量任务实例，构成任务实例池。具体细节可参考 2.3 *SkillGenesis* 中的 *Skill Proposal Stage*。**关于训练细节**：实际上我们已在补充材料 B（Training Details）中提供了相关信息，包括训练超参数、steps、mini batch size等内容。SkillEvo 框架的训练成本主要来源于两个阶段：WebGRPO 的轨迹级优化训练与 SkillGenesis 的技能演化探索。在训练阶段，WebGRPO 使用 8×H100 GPU 共训练约 38 小时，SkillGenesis 的探索阶段进行了 160 次迭代，耗时约 2.5 小时，总训练 GPU 时数约为 324 GPU小时。在查询成本方面，WebGRPO 使用 RXERM（Reasoning & Execution Reward Model）对轨迹进行奖励建模，每条轨迹需要 1+T 次 GPT-4o 调用（其中 T 为推理步数，包含 T 次推理评估与 1 次执行评估）；SkillGenesis 阶段每 20 次迭代触发一次 GPT-4o 技能组合调用。根据整体流程估算，总调用量约为 10.1 万次 GPT-4o 调用。其中RXERM两阶段奖励评估累计约 248M tokens（执行奖励约220M tokens、推理奖励约 28M tokens）。额外开销主要由 SkillGenesis 带来约42Mtokes（技能提案一次性约 64k tokens、技能验证约  42M tokens、技能演化约50k tokens）。

## **缺点5 ：缺乏失败案例分析**

感谢审稿人提出关于失败案例分析的宝贵建议。我们目前尚未在论文中系统性地讨论失败任务，这是我们后续工作的重点方向之一。未来版本中，我们计划补充典型失败案例的归因分析，例如技能调用缺失、参数填充错误，以进一步揭示方法在复杂任务下的潜在局限，并为技能结构优化和策略泛化提供方向性改进。

根据当前的初步分析，我们发现经过WebGRPO训练后，模型在部分任务上的**技能调用能力**出现下降，反映出策略执行与技能调用之间的割裂。即便在探索阶段能够正确识别合适的技能，模型在实际执行过程中未能调用，而是依赖页面操作原子动作的执行。例如在“查看某本书的详情”任务中，模型理应调用 `search_books_by_title('...')` API，但实际并未调用。这种现象不仅是调用时机判断失误，更深层次的原因在于：当前策略模型缺乏对技能调用价值的全局评估能力，技能知识未能有效整合入策略模型，导致调用时机选择不当甚至被忽略。另一个常见问题是**参数填充错误**，模型虽然正确选择了调用的 API，但在参数构造上存在语义理解偏差，尤其在处理复杂约束时更为明显。例如在“查找包含西兰花且不含奶制品的食谱”任务中，模型错误地生成了 `search_recipes_by_ingredients('broccoli, milk-free')`，而正确调用应为 `search_recipes_by_ingredients('broccoli', exclude='dairy')`。进一步分析发现，这类错误多源于模型在理解细粒度指令约束方面仍存在不足，无法正确解析否定条件和排除性约束，从而在技能参数化过程中发生信息丢失或误解。

我们希望上述补充和澄清能够解决审稿人的疑惑，如果您能提高评分我们将更加感激。

# Rebuttal to Reviewer 1HdP

我们感谢审稿人对本文的认真阅读和宝贵建议。以下针对审稿人提出的各项**缺点（Weaknesses）/问题**逐一回应。

## **缺点1:奖励机制 RXERM 没有得到充分评估**

感谢审稿人对 RXERM 奖励机制评估的关注。我们澄清，RXERM 包含两个部分：执行奖励（判断任务是否完成）和推理奖励（评估中间推理步骤的质量）。我们在早期已对 GPT‑4o 作为评估器进行了初步验证。具体而言，我们在多个任务上随机抽取了 100 条轨迹，将 GPT‑4o 的判断与人工标注结果以及其他评估模型（Claude、DeepSeek‑R1 等）进行了对比。结果显示，GPT‑4o 与人工标注的一致率较高（执行奖励 >90%，推理奖励 >85%），且与其他强评估器之间差异很小（<3%）。基于这一稳定性结果，并考虑到我们拥有 GPT‑4o 的公司内部 API 可免费使用，从成本和资源角度出发，最终选择 GPT‑4o 作为 RXERM 的评估器。此外，关于 SkillGenesis 在不同 LLM 下的性能影响，我们已在补充材料 C.4 *Performance Comparison of SkillEvo with Different LLMs* 中给出了系统分析。为回应审稿人的关切，我们将在修订版中进一步补充实验：使用多个开源评估模型替换 GPT‑4o，对 RXERM 的执行奖励准确性和推理奖励可靠性进行对比分析，从而验证方法的鲁棒性。

## **缺点2:披露模型训练所用的任务。这些训练任务的指令来源于哪里？**

SFT（监督微调，Supervised Fine-Tuning）训练所使用的数据集是 WebArena-Lite 提供的一个人类标注演示数据集。该数据集由 5 位作者基于约 40 个任务原型精心设计并生成，最终构建出1,186 条高质量的专家轨迹。这些演示覆盖了WebArena-Lite的5个真实网站，任务类型多样，操作步骤清晰

对于强化学习阶段的训练数据，正文第 3.1 节的 Baselines 部分已有明确描述。我们采用了自动任务生成方式，具体来说，指令来自于一个技能提议器（skill proposer）。该模块在训练过程中会基于初始网页状态（包含5个网站）自动生成大量任务实例，构成任务实例池。具体细节可参考 2.3 *SkillGenesis* 中的 *Skill Proposal Stage*。

## **缺点3:SkillGenesis 的技能的定义和关于SPG的细节**

在 SkillGenesis 框架中，“技能（Skill）”指的是 可复用的可执行 Python 程序，它将多个底层网页交互动作（如点击、输入、滚动等）抽象为一个高层次函数调用（higher-level function call）

技能库存储“可执行的技能函数（技能单元）”，而SPG存储的是技能之间的结构关系，包括调用顺序与依赖路径。

实际上，SPG的定义在2.3 SkillGenesis的SkillGenesisStage.中已经明确说明，SPG是用于组织技能之间结构与依赖的有向图结构，其中每个节点表示一个技能，每条边表示技能之间的调用顺序或依赖关系。其结构以 JSON 存储，由两个核心字段构成：`nodes` 和 `edges`。`nodes` 表示技能集合`edges` 表示技能之间的结构关系，记录技能之间的两种关键联系：在这两类关系中，“依赖关系” (`depends_on`) 是强约束，表示目标技能的执行在语义或状态上必须依赖源技能完成，例如：“SearchItem” 依赖用户已经完成 “Login” 操作才能执行；而“调用顺序” (`invokes`) 是弱约束，表示推荐的执行顺序，如“SearchItem → AddToCart”，但并不意味着“SearchItem”必须先于“AddToCart”执行，更多用于执行路径优化。在技能进化阶段，系统基于 SPG 中的路径结构和技能库由LLM提出技能组合建议，生成复合技能（如组合“Login”、“SearchItem”、“AddToCart”构造“OneClickPurchase”），将其作为新节点加入 `nodes`，并更新 `edges` 建立新的结构关系，从而实现技能的自动演化与调用优化，显著提升智能体的泛化能力与复杂任务的执行效率。具体可以查看2.3 SkillGenesis的 SkillEvolvingStage.。

SPG 用 JSON 表示，

```json
{
  "nodes": {
    "enter_specific_forum_section": {
      "type": "atomic",
      "description": "Navigate to the specific forum section"
    },
    "sort_forum_by_new": {
      "type": "atomic",
      "description": "Sort the forum posts by newest"
    },
    "go_to_user_profile_and_open_comments": {
      "type": "atomic",
      "description": "Open the latest post author’s profile and comments"
    },
    "count_controversial_comments": {
      "type": "atomic",
      "description": "Count comments with more downvotes than upvotes"
    }
  },
  "edges": {
    "enter_specific_forum_section": [
      {
        "target": "sort_forum_by_new",
        "type": "invokes"
      }
    ],
    "go_to_user_profile_and_open_comments": [
      {
        "target": "count_controversial_comments",
        "type": "depends_on"
      }
    ]
  }
}

```

## **缺点4:表 2 和第 5.2 节**

表 2 和第 5.2 节的结果来源于我们设计的 扩展网页浏览任务（Scaled-Up Browsing Tasks），任务列表和检查点定义详见补充材料 D Scaled-Up Tasks。这些任务运行在五个网站（Shopping、Admin、Reddit、GitLab、Map）中。WebArena -lite基准测试主要以孤立的、单一任务场景为特色，例如将某个产品添加到购物车中。然而，在现实生活中，人们通常需要连续完成一系列此类任务，例如在结账前将多件相关商品（例如咖啡和杯子）一起添加到购物车中。这类浏览请求会导致任务时间跨度极长，有时还会涉及重复的中间流程。通过在这些时间跨度长且涉及多个子任务的扩展任务我们可以展示我们可复用技能的高效性和有效性。因此与表 1 的原子性任务不同，扩展任务通过将多个相关子目标串联为一个长链任务，更接近真实用户行为。因此，表 2 与表 1 属于互补而非重复的基准。

为了准确评估这些复杂任务的执行效果，我们借鉴 Xu et al. (2024) 的做法，在每个任务设计了中间检查点，并以检查点完成比例计算成功率。这一设计可以更细粒度地衡量代理在长链任务中多阶段的稳定性，因此 SkillEvolve + SPG 在部分任务上能够达到 100% 成功率，并不意味着任务简单，而是表明其能够完整、稳定地完成长链任务中的所有子目标。

此外，扩展任务特别适合评估长期决策能力。一方面，高成功率表明代理具备在长链任务中稳定完成所有子任务的能力；另一方面，步数减少是长期决策能力的另一重要体现：能够合理地拆解复杂任务、有效复用已学技能并减少冗余探索，从而以更优路径完成任务。如果代理缺乏长期规划能力，往往会在中间阶段反复尝试或偏离目标，导致步骤增加或任务失败。SkillEvolve 在扩展任务上显著减少步骤并保持高成功率，说明其不仅能够完成任务，还能以更高效、更条理化的方式完成，真实体现了长期决策能力的提升。在这一点上，表 1 的短任务难以全面体现“跨子任务的整体规划效率”，而表 2 的扩展任务能够清晰地区分不同方法在任务拆解、路径规划与技能复用上的能力差异。SkillEvolve 在扩展任务上同时取得更高成功率与更少步骤，体现了其在真实复杂网页场景中的长期决策优势。

## **框架问题6:SkillGenesis 如何与 WebGRPO 一起使用？**

SkillGenesis 与 WebGRPO 的结合体现在 SkillEvo 框架的两阶段设计中：首先，WebGRPO 通过轨迹级强化学习训练出具备高质量推理与网页交互能力的策略模型，用于解决复杂网页环境中的多步任务；随后，在SkillGenesis 阶段，加载该策略模型，通过SkillProposal、SkillGenesis 和 SkillEvolving 三个子阶段，生成并演化可复用技能，构建技能库与技能路径图（SPG），从而显著提升模型在基准任务上的执行效率与泛化能力。

## **框架问题9:在预测中间推理奖励时，是否将整个轨迹中的所有中间推理语句一起输入模型？**

在预测中间推理奖励时，我们并不是将整条轨迹中的所有中间推理语句一次性输入模型统一评估，而是对每条推理语句进行逐条、独立的评分评估。用于该评估的具体 prompt 已在补充材料 F 节的 Figure10 *Reasoning Quality Evaluation Prompt* 中详细列出。

## **实验问题1:表 1 中的相对提升数值是基于哪些值计算的？**

感谢审稿人的评论。表1中所列“相对提升”指标（绿色行）是相对于未使用技能复用的WebGRPO（即WebGRPO without Skills）计算的。这一点在第4节“Main Results”（第246–248行，第250-252行）中已有明确说明，例如第246–248行指出，在Gitlab和Reddit环境中，相较于不使用技能复用的WebGRPO，成功率分别提升了18.0%和14.0%。我们将在表格注释中补充说明，以进一步增强表意清晰度。

## **实验问题4:第 5.2 节中实验数据来源澄清**

在第 5.2 节中提到的“1.3”和“4.9 步数”指的是在引入 SPG（Skill Planning Graph）后，SkillEvolver 在不同任务中相较于未使用 SPG 配置所减少的操作步数区间。根据表格数据，最小的步数减少值为 1.3 步，出现在 Map 任务中（从 4.5 步降至 3.2 步）；最大的步数减少值为 4.9 步，出现在 GitLab 任务中（从 20.7 步降至 15.8 步）。这两个值即为“1.3 到 4.9 步数”的具体来源。此外，Admin 任务的步数减少了 1.5 步（从 10.8 降至 9.3），Reddit 任务的步数减少了 3.4 步（从 9.9 降至 6.5）。

# Rebuttal to Reviewer 8pAH

我们感谢审稿人对本文的认真阅读和宝贵建议。以下针对审稿人提出的各项**缺点（Weaknesses）/问题**逐一回应。

## **缺点1:结果统计显著性存疑**

我们感谢审稿人对统计显著性问题的关注。我们之所以选用 WebArena 基准中的 165 个高质量子集任务进行评估，主要基于以下两个考虑：首先，原始数据集中存在部分标注错误和评估标准不明确的情况，可能会影响评估的公正性与可信度。相比之下，该子集经过人工审校与修正，能够在更大程度上保证评估结果的准确性与一致性；其次，考虑到在大规模交互式环境中进行实验的资源与时间成本，我们在保证评估质量的前提下，选择了该子集作为权衡方案。

## **缺点2:技能提议阶段的任务质量存疑**

感谢审稿人的宝贵意见。针对“技能提案阶段任务质量存疑”这一疑虑，我们在补充材料 F 中给出的 Skill Proposal Prompt 已做了专门设计：该提示词为每个候选任务设定了类型限定、交互步数上限（≤10 步）与“创建-编辑-检索”三大优先级，同时强制禁止单击式和依赖脆弱结构的技能，从源头保证了任务的覆盖面与必要复杂性；随后又采用“实用性 + 可泛化性 + 步数”的三维评分体系对技能候选进行量化评估，自动淘汰粒度过粗或过细的样例。我们认为理想的技能粒度应具有清晰边界、包含多步有效交互，并具备良好的复用性和组合潜力。值得补充的是，系统在将新技能加入任务池之前，会第一时间检测是否与现有技能重复，并自动剔除，从而保证技能库的多样性与高质量。借助 GPT-4o 卓越的指令遵循能力，这些机制在实践中得以稳定可靠地执行。关于技能粒度的动态变化，我们的框架设计确实考虑了这一点。Skill Evolving阶段专门负责技能的组合与演化，将已有的细粒度技能组合成更复杂的复合技能，系统每隔Q=20次迭代触发一次技能组合。

## **缺点3:技能进化阶段的必要性**

在完整的 165 个测试任务中，我们统计了技能调用的覆盖率、调用次数与任务复杂度的分布。经过 160 次探索迭代并在每 20 次 Genesis 迭代触发一次组合技能生成，最终技能库规模达到134  个技能，其中 基本技能88 个、组合技能 46 个（组合技能占比 34%）。测试集中共有 86 个技能被实际调用（覆盖率 64%），其中基本技能 67个、组合技能 19个。

技能调用总次数为 193 次（约 1.17 次/任务），其中基本技能调用次数 141次（73%），组合技能调用次数 52 次（27%）。组合技能虽然数量较少，但平均调用次数达到 2.74 次/技能，高于基本技能的2.10次/技能，显示出更高的复用率。

按任务级调用技能数统计：33% 的任务（55个）完全依赖原子操作即可完成，无需调用任何技能；36% 的任务（60 个）仅调用 1 个技能；调用 2 个技能和 3 个技能的任务分别占 16%（27个）和8%（13个）；调用 4个技能的高复杂度任务占 6%（10个任务）。多数简单任务可以依靠原子操作或少量基本技能完成，而高复杂度任务（≥3 次技能调用）显著依赖技能调用，贡献了  41% 的所有调用次数。

**GLM‑4‑9B（9B）**

| 训练配置                  | Reddit | GitLab | CMS  | Map  | OSS  | Avg  |
| ------------------------- | ------ | ------ | ---- | ---- | ---- | ---- |
| WebGRPO（no skills）      | 62.3   | 52.1   | 52.9 | 45.3 | 46.9 | 50.3 |
| WebGRPO（Base-only）      | 66.8   | 55.8   | 54.1 | 47.5 | 48.7 | 54.6 |
| WebGRPO（Base + Compose） | 71.2   | 61.2   | 59.2 | 50.6 | 51.5 | 57.6 |

**Llama‑3.1‑8B（8B）**

| 训练配置                  | Reddit | GitLab | CMS  | Map  | OSS  | Avg  |
| ------------------------- | ------ | ------ | ---- | ---- | ---- | ---- |
| WebGRPO（no skills）      | 65.8   | 51.3   | 59.1 | 42.3 | 44.5 | 51.8 |
| WebGRPO（Base-only）      | 69.7   | 55.2   | 61.3 | 44.1 | 49.4 | 55.9 |
| WebGRPO（Base + Compose） | 75.8   | 60.2   | 65.3 | 48.1 | 53.9 | 60.4 |

实验结果显示，GLM‑4‑9B 的平均成功率从“无技能库”的 50.3% 提升到仅引入基本技能的 54.6%（+4.3 个百分点），而在此基础上加入组合技能后进一步提升至 57.6%（+3.0 个百分点）；Llama‑3.1‑8B 的平均成功率从 51.8% → 55.9%（+4.1 个百分点），再提升到 60.4%（+4.5 个百分点）。可见，组合技能单独贡献了整体增益的约 40–50%，其提升幅度与基本技能相当甚至更高。虽然组合技能仅占技能库 34%（46/134），调用占比也仅 27%（52/193），但其平均复用频次 2.74 次/技能 显著高于基本技能（2.10 次/技能），并且在高复杂度任务中发挥关键作用。因此，技能进化阶段（组合技能的生成与使用）是保证复杂任务成功率提升的必要机制。

## **问题1多维度奖励设计引入的额外成本**

主要的额外成本来源于GPT-4o的调用，其中RXERM两阶段奖励评估累计约 248M tokens（执行奖励约220M tokens、推理奖励约 28M tokens）。额外开销主要由 SkillGenesis 带来约42Mtokes（技能提案一次性约 64k tokens、技能验证约  42M tokens、技能演化约50k tokens）。

## **问题2图5奖励组合解释**

关于如何评估奖励请你查看补充材料F中的Figure 10和11。为了让图5的奖励组合的有效性更加直观，我们以实际任务“Tell me the number of reviews that our store received by far that mention term ‘best’”为例，展示各奖励的评估维度及其对模型行为的影响。

在训练过程中的第一轮（Round 1）交互中，模型生成的输出如下所示：

<think>

让我帮你找找提到 “best” 的评论数量。当前页面是仪表盘，没看到评论的直接入口。我们需要导航到评论区域。  根据菜单结构，评论一般在 MARKETING（营销） 部分下。  让我点击 MARKETING 菜单： click("226")  点击后，应该会看到包含“评论”的子菜单。从那里我们可以搜索“best”的相关评论。这样能更准确地得出提及该词的评论数量。

</think>

<answer>

do(action="Click", element="226")

</answer>

在本轮中，模型尚未完成最终任务，但已经展示出结构合理的中间规划与有效行为。我们结合三类奖励逐一分析如下：

1.执行奖励Rexec：该轮尚未成功获取评论数量，因此执行奖励记为 0.

2.推理奖励Rreason：该奖励由大语言模型（GPT-4o）通过专设评分模板评估 `<think>` 部分的逻辑性、任务相关性、网页状态理解及中间步骤清晰度。对上述输出的评分结果为 0.975，评分说明如下：该推理非常贴近任务目标，展示了对 Magento 后台结构的良好理解，并合理规划了从“MARKETING”菜单进入“Reviews”模块以搜索关键词“best”的步骤，具有明确的中间目标和强逻辑一致性。唯一的轻微扣分在于对网页状态的理解部分：虽然推断“评论”在“MARKETING”下是合理的，但没有直接确认“Reviews”菜单项确实存在，而是基于经验进行了预判。因此在“Understanding of Webpage State”项扣了0.1，其余各项为满分，最终得分为0.975。

3.格式奖励：该输出严格遵守我们设定的 `<think>...</think><answer>...</answer>` 格式结构，符合结构约束，故格式奖励得分为 1。

此示例反映出，在真实训练过程中，许多 trajectory 尽管尚未完成任务，却具备高质量的推理过程和合理的动作选择。若仅依赖最终执行成功与否进行奖励评估（即只使用 Rexec），这类高潜力样本将被视为无效，难以对策略学习提供有效梯度。通过引入 Rreason 和 Rformat，我们能保留并强化这些“推理有效、执行未完”的中间轨迹，从而在训练初期提供稠密、稳定的监督信号，显著缓解多轮网页任务中的“稀疏奖励”与“推理退化”问题。

## **问题3为训练合成了多少任务？** 随着训练样本数量的增加，智能体的性能是如何变化的？

为训练合成了约4k个原始任务实例，160个用于SkillGenesis探索。性能随样本量先快速提升，后增益递减并最终趋于饱和。

## **问题4第5.2节关于规模化活动的分析**

感谢审稿人的细致指出。确实，文中 “SkillEvolver” 是在第 5.2 节首次正式命名，前文虽未显式使用该术语，但确实指代我们提出的方法整体。我们承认在表述上存在一定笔误和不够清晰之处，特此澄清：SkillEvolver 同时包含训练与技能学习两个核心阶段。

为分析这两个部分各自对最终结果的贡献，我们补充了如下消融实验结果：

- SkillEvolver（无 SPG & 无 skills）：100.0 % / 13.8，92.2 % / 12.7，57.0 % / 10.9，56.2 % / 23.5，100.0 % / 5.9
- SkillEvolver（无 SPG & 有 skills）：100.0 % / 10.2，93.0 % / 10.8，60.0 % / 9.9，57.8 % / 20.7，100.0 % / 4.5
- SkillEvolver（有 SPG & 有 skills）：100.0 % / 6.1，96.2 % / 9.3，64.7 % / 6.5，60.1 % / 15.8，100.0 % / 3.2

从结果可以看出，技能学习 相比无技能时显著减少步数并提升成功率，而进一步引入 SPG 则在此基础上再次提升成功率并大幅减少步数。这里需要特别说明：步数越小代表效率越高，因此 “w/ SPG” 并非性能下降，而是取得了更高的效率与成功率。可能是审稿人您在阅读表 2 时误将步数减少理解为性能降低，特此澄清。

# Rebuttal to Reviewer c6aF

我们感谢审稿人对本文的认真阅读和宝贵建议。以下针对审稿人提出的各项**缺点（Weaknesses）/问题**逐一回应。

## 缺点1：关于WebGRPO

**关于 WebGRPO：** 尽管从表 1 和图 6 来看，WebGRPO 在不使用双重不确定性主动学习时似乎不如 WebRL，但需要指出的是，**双重不确定性机制是专门针对 GRPO 优化过程中存在的问题设计的，无法直接应用于 WebRL**。具体来说，GRPO 不依赖价值网络。它在训练中容易出现这样的问题：当同一任务实例的所有 rollout 奖励相似（例如全部成功或全部失败）时，组内优势会整体退化为零，导致策略梯度（policy gradient）为 0，从而缩小梯度幅度、提高批量梯度对噪声的敏感性，进而降低采样效率。双重不确定性机制正是为了解决这一问题而设计的：通过同时考虑执行不确定性和推理不确定性稳定优势信号。相比之下，WebRL 采用的是标准的 **PPO/actor–critic 框架**，具有价值网络并基于**动作级奖励**优化，本身并不存在组内优势塌缩的问题。因此，直接将双重不确定性机制迁移到 WebRL 上既缺乏理论适配性，也难以带来同样的效果。出于同样原因，RXERM（Reasoning & Execution Reward Model）也是为WebGRPO 设计的轨迹级奖励建模方式，用于为 GRPO 提供更细粒度的推理与执行奖励信号，而不是简单替换 WebRL 的奖励函数。可以说，RXERM 与双重不确定性机制共同构成了一套专门稳定 **GRPO 轨迹级策略优化**的整体方案，而并非面向 PPO 类算法的通用插件。

## 缺点2：相关成本

主要的额外成本来源于GPT-4o的调用，其中RXERM两阶段奖励评估累计约 248M tokens（执行奖励约220M tokens、推理奖励约 28M tokens）。额外开销主要由 SkillGenesis 带来约42Mtokes（技能提案一次性约 64k tokens、技能验证约  42M tokens、技能演化约50k tokens）。

## 缺点3：训练数据

SFT（监督微调，Supervised Fine-Tuning）训练所使用的数据集是 WebArena-Lite 提供的一个人类标注演示数据集。该数据集由 **5 位作者**基于约 **40 个任务原型** 精心设计并生成，最终构建出 **1,186 条高质量的专家轨迹（expert demonstrations）**。这些演示覆盖了WebArena-Lite的5个真实网站，任务类型多样，操作步骤清晰。对于强化学习阶段的训练数据，正文第 3.1 节的 Baselines 部分已有明确描述。我们采用了自动任务生成方式，具体来说，指令来自于一个技能提议器（skill proposer）。该模块在训练过程中会基于初始网页状态（包含5个网站）自动生成大量任务实例，构成任务实例池。具体细节可参考 2.3 *SkillGenesis* 中的 *Skill Proposal Stage*。

## 缺点4：创新性

我们不同意将本工作视为 WebRL 和 ASI 的简单组合。WebRL  虽然提出了 curriculum 与 RL 结合的思路，但采用的是基于 PPO 的 RL，需要额外的 critic 模型和复杂的训练机制，主要优化动作级别奖励，难以解决长序列推理和稀疏奖励问题；ASI 虽然将技能抽象为可执行程序并纳入动作空间以提升效率，但依赖静态提示驱动技能更新，使其对新任务和多样化网页环境的适应性和通用性较差。针对这些不足，我们提出了一个双阶段训练框架，包括：轨迹级 GRPO (WebGRPO) 将 GRPO 扩展到完整推理–交互轨迹并直接优化轨迹级策略；RXERM 双阶段奖励机制解耦推理与执行评价，系统性地解决了推理能力难以自然涌现的关键挑战；双重不确定性主动学习提升样本信息量和训练稳定性；以及 SkillGenesis 技能进化框架（SPG 驱动）动态抽取、组合和进化技能以增强跨任务泛化。这一“轨迹级优化 + 技能级进化”的闭环设计兼顾稳定性与泛化性，效果显著，在 WebArena-Lite 上成功率绝对提升达18%（Llama3.1+WebGRPO(+Skills)的60.4% vs. Llama3.1+WebRL 的 42.4%），充分体现了方法的新颖性与有效性。

## 问题1：R_reason评分

感谢审稿人的细致提问。关于第145行，我们确实对每个 `<think>` 推理单元进行独立评分，并将其平均以计算整体的 R_reason

## 问题2：训练数据

SFT（监督微调，Supervised Fine-Tuning）训练所使用的数据集是 WebArena-Lite 提供的一个人类标注演示数据集。该数据集由 **5 位作者**基于约 **40 个任务原型** 精心设计并生成，最终构建出 **1,186 条高质量的专家轨迹（expert demonstrations）**。这些演示覆盖了WebArena-Lite的5个真实网站，任务类型多样，操作步骤清晰。对于强化学习阶段的训练数据，正文第 3.1 节的 Baselines 部分已有明确描述。我们采用了自动任务生成方式，具体来说，指令来自于一个技能提议器（skill proposer）。该模块在训练过程中会基于初始网页状态（包含5个网站）自动生成大量任务实例，构成任务实例池。具体细节可参考 2.3 *SkillGenesis* 中的 *Skill Proposal Stage*。

## 问题3：关于SkillGenesis

在完整的 165 个测试任务中，我们统计了技能调用的覆盖率、调用次数与任务复杂度的分布。经过 160 次探索迭代并在每 20 次 Genesis 迭代触发一次组合技能生成，最终技能库规模达到134  个技能，其中 基本技能88 个、组合技能 46 个（组合技能占比 34%）。测试集中共有 86 个技能被实际调用（覆盖率 64%），其中基本技能 67个、组合技能 19个。

技能调用总次数为 193 次（约 1.17 次/任务），其中基本技能调用次数 141次（73%），组合技能调用次数 52 次（27%）。组合技能虽然数量较少，但平均调用次数达到 2.74 次/技能，高于基本技能的2.10次/技能，显示出更高的复用率。

按任务级调用技能数统计：33% 的任务（55个）完全依赖原子操作即可完成，无需调用任何技能；36% 的任务（60 个）仅调用 1 个技能；调用 2 个技能和 3 个技能的任务分别占 16%（27个）和8%（13个）；调用 4个技能的高复杂度任务占 6%（10个任务）。多数简单任务可以依靠原子操作或少量基本技能完成，而高复杂度任务（≥3 次技能调用）显著依赖技能调用，贡献了  41% 的所有调用次数。

在本实验中，技能集的访问与筛选通过 Prompt 内直接列出技能函数 的方式完成。具体地，将 134 个技能函数平均分为 5 批，每批约 27 个函数，并嵌入到 Prompt 的 `<functions>...</functions>` 区块中。模型在每一批 Prompt 内基于任务描述逐一判断技能是否相关，并输出包含 `step by step reasoning` 与 `function names` 的 JSON 结果。随后将 5 批的结果合并去重，得到候选技能集，再运行一轮排序 Prompt 按相关性筛选出 Top-15 技能函数，最终将这些技能注入策略模型 Prompt 中执行任务。与数据库检索相比，该方法更加直接和可解释。一方面，技能库规模较小，分批直接列入 Prompt 足以覆盖所有技能，不必依赖外部检索系统；另一方面，数据库或向量检索可能因召回不精确而漏掉关键技能，而直接列出技能则可确保完整性和透明性。此方法能减少实验变量，保证筛选逻辑完全由模型在 Prompt 内完成，便于验证技能筛选对任务执行性能的影响。

```SQL
A.8 Skill Retrieval

You are provided with a list of Python functions that represent action shortcuts available on a website (in addition to basic actions like click, type, hover, select option, etc.).

<functions>
{function space}
</functions>

Your task is:
{repr task}

Please analyze each function and determine whether it is useful for the task.  
Finally, output a JSON in the following format:
{
  "step by step reasoning": "Explain your reasoning process",
  "function names": ["function_name_1", "function_name_2", "function_name_3"]
}

```



```Python
Now you have a list of candidate functions :

{candidate functions list}

Your task is:
{repr task}

Please rank these functions based on their relevance to the task 
and return the Top-15 most relevant functions.

Output JSON in the following format:
{
  "step by step reasoning": "Explain how you determined the ranking",
  "function names": ["Top1_function_name", "Top2_function_name", ..., "Top15_function_name"]
}
```