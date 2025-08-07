# Repositório de Helm Charts

Este repositório contém uma coleção de Helm Charts reutilizáveis e prontos para produção, projetados para padronizar e acelerar o deploy de aplicações no Kubernetes.

## 🎯 Propósito

O objetivo deste projeto é centralizar a lógica de deploy em charts bem mantidos e documentados, seguindo o princípio DRY (Don't Repeat Yourself). Em vez de cada serviço reinventar seus manifestos, eles podem usar estes charts como uma dependência, garantindo consistência em toda a organização.

📦 Charts Disponíveis

A seguir, uma lista dos charts disponíveis neste repositório.

| Chart      | Versão | Descrição                                                                                             | Documentação         |
|------------|--------|--------------------------------------------------------------------------------------------------------|----------------------|
| base-app   | 1.1.0  | Um chart genérico e altamente configurável para deploy de qualquer aplicação, suportando Deployments, StatefulSets, Argo Rollouts e External Secrets. | [README](./charts/base-app/README.md) |

🤝 Contribuição

Sinta-se à vontade para abrir issues ou pull requests para sugerir melhorias ou adicionar novos charts. Toda contribuição é bem-vinda!

📄 Licença

Este projeto é distribuído sob a licença MIT.
