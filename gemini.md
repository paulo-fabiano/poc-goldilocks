Atue como um Engenheiro de Software SRE especialista em Kubernetes. Preciso criar uma Custom Resource Definition (CRD) em YAML para um projeto de FinOps e Otimização de Recursos.

O objetivo da CRD: Armazenar o relatório de análise gerado por um script customizado que consome dados do Goldilocks e do VPA.

Especificações da CRD:

Nome: EfficiencyReport (plural: efficiencyreports).

Group: finops.k8s.io.

Version: v1alpha1.

Escopo: Namespaced.

Campos obrigatórios no Schema (Validation):

targetRef: Nome do Deployment/StatefulSet analisado.

currentResources: CPU e Memória configurados atualmente no Helm/Manifesto.

recommendedResources: CPU e Memória recomendados pelo Goldilocks (Target).

savingsPotential: Um valor numérico ou string indicando a porcentagem de economia (ex: "45%").

status: Um campo de status que pode ser 'Optimized', 'Overprovisioned' ou 'Underprovisioned'.

Requisitos Adicionais:

Inclua a seção additionalPrinterColumns para que, ao dar um kubectl get efficiencyreports, eu consiga ver colunas customizadas como 'DEPLOYMENT', 'SAVINGS' e 'STATUS' diretamente no terminal.

Gere o YAML seguindo as melhores práticas de OpenAPI v3 para validação dos campos."